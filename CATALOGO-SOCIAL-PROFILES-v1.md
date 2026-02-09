# CATALOGO-SOCIAL-PROFILES-v1

> **Dominio**: Social
> **Stack**: Next.js, React, TypeScript, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

§ 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [prisma/schema-social.prisma (250 righe)](#1-prisma-schema-social-prisma-(250-righe)) | `prisma/schema-social.prisma (250 righe)` |
| 2 | [src/server/services/profile-service.ts (250 righe)](#2-src-server-services-profile-service-ts-(250-righe)) | `src/server/services/profile-service.ts (250 righe)` |
| 3 | [src/server/services/post-service.ts (300 righe)](#3-src-server-services-post-service-ts-(300-righe)) | `src/server/services/post-service.ts (300 righe)` |
| 4 | [src/server/services/feed-service.ts (200 righe)](#4-src-server-services-feed-service-ts-(200-righe)) | `src/server/services/feed-service.ts (200 righe)` |
| 5 | [src/server/services/follow-service.ts (150 righe)](#5-src-server-services-follow-service-ts-(150-righe)) | `src/server/services/follow-service.ts (150 righe)` |
| 6 | [src/server/trpc/routers/social.ts (250 righe)](#6-src-server-trpc-routers-social-ts-(250-righe)) | `src/server/trpc/routers/social.ts (250 righe)` |
| 7 | [src/lib/validations/social.ts (100 righe)](#7-src-lib-validations-social-ts-(100-righe)) | `src/lib/validations/social.ts (100 righe)` |
| 8 | [src/hooks/use-profile.ts (80 righe)](#8-src-hooks-use-profile-ts-(80-righe)) | `src/hooks/use-profile.ts (80 righe)` |
| 9 | [src/hooks/use-posts.ts (100 righe)](#9-src-hooks-use-posts-ts-(100-righe)) | `src/hooks/use-posts.ts (100 righe)` |
| 10 | [src/hooks/use-feed.ts (80 righe)](#10-src-hooks-use-feed-ts-(80-righe)) | `src/hooks/use-feed.ts (80 righe)` |
| 11 | [src/hooks/use-follow.ts (60 righe)](#11-src-hooks-use-follow-ts-(60-righe)) | `src/hooks/use-follow.ts (60 righe)` |
| 12 | [src/components/social/profile-card.tsx (100 righe)](#12-src-components-social-profile-card-tsx-(100-righe)) | `src/components/social/profile-card.tsx (100 righe)` |
| 13 | [src/components/social/profile-header.tsx (150 righe)](#13-src-components-social-profile-header-tsx-(150-righe)) | `src/components/social/profile-header.tsx (150 righe)` |
| 14 | [src/components/social/post-card.tsx (200 righe)](#14-src-components-social-post-card-tsx-(200-righe)) | `src/components/social/post-card.tsx (200 righe)` |
| 15 | [src/components/social/post-composer.tsx (200 righe)](#15-src-components-social-post-composer-tsx-(200-righe)) | `src/components/social/post-composer.tsx (200 righe)` |
| 16 | [src/components/social/feed.tsx (100 righe)](#16-src-components-social-feed-tsx-(100-righe)) | `src/components/social/feed.tsx (100 righe)` |
| 17 | [src/components/social/follow-button.tsx (80 righe)](#17-src-components-social-follow-button-tsx-(80-righe)) | `src/components/social/follow-button.tsx (80 righe)` |
| 18 | [src/components/social/user-list.tsx (80 righe)](#18-src-components-social-user-list-tsx-(80-righe)) | `src/components/social/user-list.tsx (80 righe)` |
| 19 | [src/app/(social)/feed/page.tsx (60 righe)](#19-src-app-(social)-feed-page-tsx-(60-righe)) | `src/app/(social)/feed/page.tsx (60 righe)` |
| 20 | [src/app/(social)/explore/page.tsx (60 righe)](#20-src-app-(social)-explore-page-tsx-(60-righe)) | `src/app/(social)/explore/page.tsx (60 righe)` |
| 21 | [src/app/(social)/[username]/page.tsx (100 righe)](#21-src-app-(social)-username-page-tsx-(100-righe)) | `src/app/(social)/[username]/page.tsx (100 righe)` |
| 22 | [src/app/(social)/post/[id]/page.tsx (80 righe)](#22-src-app-(social)-post-id-page-tsx-(80-righe)) | `src/app/(social)/post/[id]/page.tsx (80 righe)` |
| 23 | [tests/social.test.ts (200 righe)](#23-tests-social-test-ts-(200-righe)) | `tests/social.test.ts (200 righe)` |

---

§ 1. PRISMA/SCHEMA-SOCIAL.PRISMA (250 RIGHE)
prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql" // Or 'sqlite', 'mysql', etc.
  url      = env("DATABASE_URL")
}

// Minimal User model, assuming authentication is handled externally
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  image         String?
  profile       Profile?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Profile {
  id             String     @id @default(cuid())
  userId         String     @unique
  user           User       @relation(fields: [userId], references: [id], onDelete: Cascade)

  username       String     @unique
  displayName    String
  bio            String?    @db.Text
  avatar         String?
  coverImage     String?
  website        String?
  location       String?

  // Stats (denormalized for performance)
  followersCount Int        @default(0)
  followingCount Int        @default(0)
  postsCount     Int        @default(0)

  // Privacy
  isPrivate      Boolean    @default(false)
  isVerified     Boolean    @default(false)

  // Relations
  posts          Post[]
  comments       Comment[]
  likes          Like[]
  bookmarks      Bookmark[]
  followers      Follow[]   @relation("followers") // Users who follow this profile
  following      Follow[]   @relation("following") // Users this profile follows
  notifications  Notification[] @relation("recipient")
  sentNotifications Notification[] @relation("sender")

  createdAt      DateTime   @default(now())
  updatedAt      DateTime   @updatedAt

  @@index([username])
  @@index([userId])
}

model Post {
  id            String         @id @default(cuid())
  authorId      String
  author        Profile        @relation(fields: [authorId], references: [id], onDelete: Cascade)

  content       String         @db.Text
  // Media
  images        String[]       // Array of URLs
  videos        String[]

  // Engagement stats (denormalized)
  likesCount    Int            @default(0)
  commentsCount Int            @default(0)
  sharesCount   Int            @default(0) // For external shares or internal reposts
  viewsCount    Int            @default(0)

  // Status
  visibility    PostVisibility @default(PUBLIC)
  isPinned      Boolean        @default(false)

  // Repost/Quote
  repostOfId    String?
  repostOf      Post?          @relation("reposts", fields: [repostOfId], references: [id], onDelete: NoAction)
  reposts       Post[]         @relation("reposts") // Posts that are reposts of this one

  // Relations
  comments      Comment[]
  likes         Like[]
  bookmarks     Bookmark[]
  notifications Notification[]

  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  deletedAt     DateTime?

  @@index([authorId])
  @@index([createdAt])
  @@index([visibility])
  @@index([repostOfId])
}

enum PostVisibility {
  PUBLIC
  FOLLOWERS_ONLY
  PRIVATE // Only author can see (drafts, personal notes)
}

model Comment {
  id        String    @id @default(cuid())
  content   String    @db.Text
  authorId  String
  author    Profile   @relation(fields: [authorId], references: [id], onDelete: Cascade)
  postId    String
  post      Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  parentId  String?   // For nested comments
  parent    Comment?  @relation("replies", fields: [parentId], references: [id], onDelete: NoAction)
  replies   Comment[] @relation("replies")

  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([authorId])
  @@index([postId])
  @@index([parentId])
}

model Follow {
  id          String       @id @default(cuid())
  followerId  String
  follower    Profile      @relation("following", fields: [followerId], references: [id], onDelete: Cascade)
  followingId String
  following   Profile      @relation("followers", fields: [followingId], references: [id], onDelete: Cascade)

  status      FollowStatus @default(ACCEPTED)

  createdAt   DateTime     @default(now())

  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}

enum FollowStatus {
  PENDING   // For private accounts
  ACCEPTED
  REJECTED
}

model Like {
  id        String   @id @default(cuid())
  profileId String
  profile   Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade)
  postId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())

  @@unique([profileId, postId])
  @@index([postId])
  @@index([profileId])
}

model Bookmark {
  id        String   @id @default(cuid())
  profileId String
  profile   Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade)
  postId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())

  @@unique([profileId, postId])
  @@index([postId])
  @@index([profileId])
}

enum NotificationType {
  LIKE
  COMMENT
  FOLLOW
  REPOST
  MENTION
}

model Notification {
  id          String         @id @default(cuid())
  recipientId String
  recipient   Profile        @relation("recipient", fields: [recipientId], references: [id], onDelete: Cascade)
  senderId    String?        // Optional, e.g., for system notifications
  sender      Profile?       @relation("sender", fields: [senderId], references: [id], onDelete: SetNull)
  type        NotificationType
  postId      String?
  post        Post?          @relation(fields: [postId], references: [id], onDelete: SetNull)
  commentId   String?
  comment     Comment?       @relation(fields: [commentId], references: [id], onDelete: SetNull)
  read        Boolean        @default(false)
  createdAt   DateTime       @default(now())

  @@index([recipientId])
  @@index([senderId])
  @@index([postId])
  @@index([createdAt])
}

---

§ 2. SRC/SERVER/SERVICES/PROFILE-SERVICE.TS (250 RIGHE)
typescript
import { PrismaClient, Profile, User } from '@prisma/client';
import { CreateProfileInput, SearchParams, UpdateProfileInput } from '~/lib/validations/social';

const prisma = new PrismaClient();

export class ProfileService {
  // CRUD
  async create(userId: string, data: CreateProfileInput): Promise<Profile> {
    const { username, displayName, bio, avatar, coverImage, website, location, isPrivate } = data;
    return prisma.profile.create({
      data: {
        userId,
        username,
        displayName,
        bio,
        avatar,
        coverImage,
        website,
        location,
        isPrivate,
      },
    });
  }

  async update(profileId: string, data: UpdateProfileInput): Promise<Profile> {
    return prisma.profile.update({
      where: { id: profileId },
      data: {
        ...data,
        updatedAt: new Date(),
      },
    });
  }

  async getById(id: string): Promise<Profile | null> {
    return prisma.profile.findUnique({
      where: { id },
      include: { user: true }, // Include user data if needed
    });
  }

  async getByUsername(username: string): Promise<Profile | null> {
    return prisma.profile.findUnique({
      where: { username },
      include: { user: true },
    });
  }

  async getByUserId(userId: string): Promise<Profile | null> {
    return prisma.profile.findUnique({
      where: { userId },
      include: { user: true },
    });
  }

  // Search
  async search(query: string, params?: SearchParams): Promise<Profile[]> {
    const { limit = 10, offset = 0 } = params || {};
    return prisma.profile.findMany({
      where: {
        OR: [
          { username: { contains: query, mode: 'insensitive' } },
          { displayName: { contains: query, mode: 'insensitive' } },
        ],
      },
      take: limit,
      skip: offset,
      orderBy: { followersCount: 'desc' }, // Simple ranking
    });
  }

  async getSuggested(profileId: string, limit: number = 5): Promise<Profile[]> {
    // A simple suggestion algorithm: find profiles followed by people you follow,
    // or popular profiles not yet followed.
    // This is a placeholder; a real algorithm would be more complex.
    const following = await prisma.follow.findMany({
      where: { followerId: profileId, status: 'ACCEPTED' },
      select: { followingId: true },
    });
    const followingIds = following.map(f => f.followingId);

    return prisma.profile.findMany({
      where: {
        id: { notIn: [...followingIds, profileId] },
        isPrivate: false,
      },
      orderBy: { followersCount: 'desc' },
      take: limit,
    });
  }

  // Username
  async isUsernameAvailable(username: string): Promise<boolean> {
    const profile = await prisma.profile.findUnique({ where: { username } });
    return profile === null;
  }

  async generateUsername(name: string): Promise<string> {
    let baseUsername = name.toLowerCase().replace(/\s/g, '');
    let username = baseUsername;
    let counter = 1;
    while (!(await this.isUsernameAvailable(username))) {
      username = `${baseUsername}${counter}`;
      counter++;
    }
    return username;
  }

  // Stats
  async updateStats(profileId: string): Promise<void> {
    const followersCount = await prisma.follow.count({
      where: { followingId: profileId, status: 'ACCEPTED' },
    });
    const followingCount = await prisma.follow.count({
      where: { followerId: profileId, status: 'ACCEPTED' },
    });
    const postsCount = await prisma.post.count({
      where: { authorId: profileId, deletedAt: null },
    });

    await prisma.profile.update({
      where: { id: profileId },
      data: {
        followersCount,
        followingCount,
        postsCount,
      },
    });
  }

  async incrementStat(profileId: string, stat: 'followers' | 'following' | 'posts', delta: number): Promise<void> {
    let data: { [key: string]: any } = {};
    if (stat === 'followers') data.followersCount = { increment: delta };
    else if (stat === 'following') data.followingCount = { increment: delta };
    else if (stat === 'posts') data.postsCount = { increment: delta };

    await prisma.profile.update({
      where: { id: profileId },
      data,
    });
  }
}

---

§ 3. SRC/SERVER/SERVICES/POST-SERVICE.TS (300 RIGHE)
typescript
import { PrismaClient, Post, Profile, PostVisibility } from '@prisma/client';
import { CreatePostInput, ListParams, PaginatedResult, UpdatePostInput } from '~/lib/validations/social';
import { ProfileService } from './profile-service';
import { NotificationService } from './notification-service'; // Assuming a notification service

const prisma = new PrismaClient();
const profileService = new ProfileService();
const notificationService = new NotificationService();

export class PostService {
  // CRUD
  async create(authorId: string, data: CreatePostInput): Promise<Post> {
    const { content, images, videos, visibility, repostOfId } = data;
    const post = await prisma.post.create({
      data: {
        authorId,
        content,
        images: images || [],
        videos: videos || [],
        visibility: visibility || PostVisibility.PUBLIC,
        repostOfId,
      },
    });

    await profileService.incrementStat(authorId, 'posts', 1);

    // Handle mentions and hashtags
    const mentions = this.extractMentions(content);
    for (const username of mentions) {
      const mentionedProfile = await profileService.getByUsername(username);
      if (mentionedProfile) {
        await notificationService.create(mentionedProfile.id, authorId, 'MENTION', post.id);
      }
    }
    // Real-time: notify followers, etc.
    return post;
  }

  async update(postId: string, data: UpdatePostInput): Promise<Post> {
    return prisma.post.update({
      where: { id: postId },
      data: {
        ...data,
        updatedAt: new Date(),
      },
    });
  }

  async delete(postId: string): Promise<void> {
    const post = await prisma.post.update({
      where: { id: postId },
      data: { deletedAt: new Date() },
      select: { authorId: true },
    });
    if (post) {
      await profileService.incrementStat(post.authorId, 'posts', -1);
    }
    // Real-time: invalidate feeds
  }

  async getById(id: string): Promise<Post | null> {
    return prisma.post.findUnique({
      where: { id, deletedAt: null },
      include: { author: true, repostOf: { include: { author: true } } },
    });
  }

  // Queries
  async getByAuthor(authorId: string, params?: ListParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    const posts = await prisma.post.findMany({
      where: { authorId, deletedAt: null, visibility: { not: PostVisibility.PRIVATE } },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { author: true, repostOf: { include: { author: true } } },
    });
    return {
      items: posts,
      nextCursor: posts.length === limit ? posts[posts.length - 1]?.id : undefined,
    };
  }

  async getByHashtag(hashtag: string, params?: ListParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    // This requires full-text search or a dedicated hashtag table for efficiency
    // For now, a simple 'contains' on content
    const posts = await prisma.post.findMany({
      where: {
        content: { contains: `#${hashtag}`, mode: 'insensitive' },
        deletedAt: null,
        visibility: PostVisibility.PUBLIC,
      },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { author: true, repostOf: { include: { author: true } } },
    });
    return {
      items: posts,
      nextCursor: posts.length === limit ? posts[posts.length - 1]?.id : undefined,
    };
  }

  // Engagement
  async like(postId: string, profileId: string): Promise<void> {
    try {
      await prisma.like.create({ data: { postId, profileId } });
      await prisma.post.update({
        where: { id: postId },
        data: { likesCount: { increment: 1 } },
      });
      const post = await this.getById(postId);
      if (post && post.authorId !== profileId) {
        await notificationService.create(post.authorId, profileId, 'LIKE', postId);
      }
      // Real-time: notify post author
    } catch (error: any) {
      if (error.code === 'P2002') {
        // Unique constraint failed, already liked
        console.warn(`Profile ${profileId} already liked post ${postId}`);
      } else {
        throw error;
      }
    }
  }

  async unlike(postId: string, profileId: string): Promise<void> {
    const deleted = await prisma.like.deleteMany({ where: { postId, profileId } });
    if (deleted.count > 0) {
      await prisma.post.update({
        where: { id: postId },
        data: { likesCount: { decrement: 1 } },
      });
      // Real-time: remove notification if it exists (complex)
    }
  }

  async isLiked(postId: string, profileId: string): Promise<boolean> {
    const like = await prisma.like.findUnique({ where: { profileId_postId: { postId, profileId } } });
    return like !== null;
  }

  async getLikers(postId: string, params?: ListParams): Promise<PaginatedResult<Profile>> {
    const { limit = 10, cursor } = params || {};
    const likes = await prisma.like.findMany({
      where: { postId },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { profile: true },
    });
    return {
      items: likes.map(l => l.profile),
      nextCursor: likes.length === limit ? likes[likes.length - 1]?.id : undefined,
    };
  }

  // Bookmarks
  async bookmark(postId: string, profileId: string): Promise<void> {
    try {
      await prisma.bookmark.create({ data: { postId, profileId } });
    } catch (error: any) {
      if (error.code === 'P2002') console.warn(`Profile ${profileId} already bookmarked post ${postId}`);
      else throw error;
    }
  }

  async unbookmark(postId: string, profileId: string): Promise<void> {
    await prisma.bookmark.deleteMany({ where: { postId, profileId } });
  }

  async getBookmarks(profileId: string, params?: ListParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    const bookmarks = await prisma.bookmark.findMany({
      where: { profileId },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { post: { include: { author: true, repostOf: { include: { author: true } } } } },
    });
    return {
      items: bookmarks.map(b => b.post),
      nextCursor: bookmarks.length === limit ? bookmarks[bookmarks.length - 1]?.id : undefined,
    };
  }

  // Repost
  async repost(postId: string, profileId: string, quote?: string): Promise<Post> {
    const originalPost = await this.getById(postId);
    if (!originalPost) throw new Error('Original post not found');

    const repost = await prisma.post.create({
      data: {
        authorId: profileId,
        content: quote || '', // Quote content if provided
        repostOfId: postId,
        visibility: PostVisibility.PUBLIC, // Reposts are usually public
      },
    });
    await prisma.post.update({
      where: { id: postId },
      data: { sharesCount: { increment: 1 } },
    });
    await profileService.incrementStat(profileId, 'posts', 1);
    if (originalPost.authorId !== profileId) {
      await notificationService.create(originalPost.authorId, profileId, 'REPOST', postId);
    }
    return repost;
  }

  async unrepost(postId: string, profileId: string): Promise<void> {
    // Find the repost created by this profile for the given postId
    const repost = await prisma.post.findFirst({
      where: { authorId: profileId, repostOfId: postId, deletedAt: null },
    });
    if (repost) {
      await prisma.post.update({ where: { id: repost.id }, data: { deletedAt: new Date() } });
      await prisma.post.update({
        where: { id: postId },
        data: { sharesCount: { decrement: 1 } },
      });
      await profileService.incrementStat(profileId, 'posts', -1);
    }
  }

  // Pin
  async pin(postId: string, authorId: string): Promise<void> {
    // Unpin any other pinned posts by this author first
    await prisma.post.updateMany({
      where: { authorId, isPinned: true },
      data: { isPinned: false },
    });
    await prisma.post.update({
      where: { id: postId, authorId },
      data: { isPinned: true },
    });
  }

  async unpin(postId: string, authorId: string): Promise<void> {
    await prisma.post.update({
      where: { id: postId, authorId },
      data: { isPinned: false },
    });
  }

  // Stats
  async incrementViews(postId: string): Promise<void> {
    await prisma.post.update({
      where: { id: postId },
      data: { viewsCount: { increment: 1 } },
    });
  }

  async updateEngagementStats(postId: string): Promise<void> {
    const likesCount = await prisma.like.count({ where: { postId } });
    const commentsCount = await prisma.comment.count({ where: { postId } });
    const sharesCount = await prisma.post.count({ where: { repostOfId: postId, deletedAt: null } });

    await prisma.post.update({
      where: { id: postId },
      data: {
        likesCount,
        commentsCount,
        sharesCount,
      },
    });
  }

  // Helpers
  private extractHashtags(content: string): string[] {
    const hashtagRegex = /#(\w+)/g;
    const matches = content.match(hashtagRegex);
    return matches ? Array.from(new Set(matches.map(tag => tag.substring(1)))) : [];
  }

  private extractMentions(content: string): string[] {
    const mentionRegex = /@(\w+)/g;
    const matches = content.match(mentionRegex);
    return matches ? Array.from(new Set(matches.map(mention => mention.substring(1)))) : [];
  }
}

// Placeholder for NotificationService
class NotificationService {
  async create(recipientId: string, senderId: string, type: string, postId?: string, commentId?: string) {
    // console.log(`Notification: ${senderId} -> ${recipientId} (${type}) on post ${postId}`);
    // In a real app, this would create a DB entry and push a real-time notification
    await prisma.notification.create({
      data: {
        recipientId,
        senderId,
        type: type as any, // Cast to NotificationType enum
        postId,
        commentId,
      },
    });
  }
}

---

§ 4. SRC/SERVER/SERVICES/FEED-SERVICE.TS (200 RIGHE)
typescript
import { PrismaClient, Post, Profile, PostVisibility, FollowStatus } from '@prisma/client';
import { FeedParams, PaginatedResult } from '~/lib/validations/social';
import { ProfileService } from './profile-service';
import { FollowService } from './follow-service';

const prisma = new PrismaClient();
const profileService = new ProfileService();
const followService = new FollowService();

export class FeedService {
  // Feed generation
  async getHomeFeed(profileId: string, params?: FeedParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    const viewerProfile = await profileService.getById(profileId);
    if (!viewerProfile) throw new Error('Viewer profile not found');

    const following = await prisma.follow.findMany({
      where: { followerId: profileId, status: FollowStatus.ACCEPTED },
      select: { followingId: true },
    });
    const followingIds = following.map(f => f.followingId);

    // Include own posts and posts from followed users
    const posts = await prisma.post.findMany({
      where: {
        authorId: { in: [...followingIds, profileId] },
        deletedAt: null,
        visibility: { in: [PostVisibility.PUBLIC, PostVisibility.FOLLOWERS_ONLY] }, // Consider privacy
      },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { author: true, repostOf: { include: { author: true } } },
    });

    const rankedPosts = await this.rankPosts(posts, viewerProfile);

    return {
      items: rankedPosts,
      nextCursor: posts.length === limit ? posts[posts.length - 1]?.id : undefined,
    };
  }

  async getExploreFeed(profileId: string, params?: FeedParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    const viewerProfile = await profileService.getById(profileId);
    if (!viewerProfile) throw new Error('Viewer profile not found');

    // Fetch popular public posts not from users the viewer follows
    const following = await prisma.follow.findMany({
      where: { followerId: profileId, status: FollowStatus.ACCEPTED },
      select: { followingId: true },
    });
    const followingIds = following.map(f => f.followingId);

    const posts = await prisma.post.findMany({
      where: {
        authorId: { notIn: [...followingIds, profileId] },
        deletedAt: null,
        visibility: PostVisibility.PUBLIC,
      },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: [{ likesCount: 'desc' }, { createdAt: 'desc' }], // Simple popularity + recency
      include: { author: true, repostOf: { include: { author: true } } },
    });

    const rankedPosts = await this.rankPosts(posts, viewerProfile);

    return {
      items: rankedPosts,
      nextCursor: posts.length === limit ? posts[posts.length - 1]?.id : undefined,
    };
  }

  async getProfileFeed(profileId: string, viewerId?: string, params?: FeedParams): Promise<PaginatedResult<Post>> {
    const { limit = 10, cursor } = params || {};
    const targetProfile = await profileService.getById(profileId);
    if (!targetProfile) throw new Error('Profile not found');

    let whereClause: any = {
      authorId: profileId,
      deletedAt: null,
    };

    if (targetProfile.isPrivate && viewerId !== profileId) {
      const isFollowing = viewerId ? await followService.isFollowing(viewerId, profileId) : false;
      if (!isFollowing) {
        // If private and not following, only show public posts (if any, though usually none for private)
        // Or, more strictly, no posts at all. Let's assume no posts for private if not following.
        return { items: [], nextCursor: undefined };
      }
      // If following a private account, show all posts except PRIVATE
      whereClause.visibility = { not: PostVisibility.PRIVATE };
    } else if (viewerId !== profileId) {
      // Public profile, but not the author viewing their own profile
      whereClause.visibility = { not: PostVisibility.PRIVATE };
    }
    // If author viewing their own profile, show all posts including PRIVATE (drafts)
    // No additional filter needed for `visibility` in this case.

    const posts = await prisma.post.findMany({
      where: whereClause,
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: [{ isPinned: 'desc' }, { createdAt: 'desc' }],
      include: { author: true, repostOf: { include: { author: true } } },
    });

    return {
      items: posts,
      nextCursor: posts.length === limit ? posts[posts.length - 1]?.id : undefined,
    };
  }

  // Algorithm (simplified for demonstration)
  private async rankPosts(posts: Post[], viewerProfile: Profile): Promise<Post[]> {
    // In a real application, this would involve more sophisticated ML models
    // and potentially external services.
    return posts.sort((a, b) => {
      const scoreA = this.calculateEngagementScore(a) + this.calculateRecencyScore(a) + this.calculateRelevanceScore(a, viewerProfile);
      const scoreB = this.calculateEngagementScore(b) + this.calculateRecencyScore(b) + this.calculateRelevanceScore(b, viewerProfile);
      return scoreB - scoreA; // Higher score first
    });
  }

  private calculateEngagementScore(post: Post): number {
    return post.likesCount * 0.5 + post.commentsCount * 0.7 + post.sharesCount * 1.0 + post.viewsCount * 0.1;
  }

  private calculateRecencyScore(post: Post): number {
    const now = new Date();
    const ageInHours = (now.getTime() - post.createdAt.getTime()) / (1000 * 60 * 60);
    return Math.max(0, 100 - ageInHours * 0.5); // Posts get less score over time
  }

  private calculateRelevanceScore(post: Post, viewer: Profile): number {
    // Placeholder: In a real system, this would consider user interests,
    // past interactions, content similarity, etc.
    // For now, give a slight boost if the author is followed.
    // This would require fetching follow status for each post author, which is inefficient here.
    // A more efficient approach would be to filter by followed authors first.
    return 0; // Simplified
  }

  // Caching (placeholder for Redis/Memcached integration)
  async invalidateFeedCache(profileId: string): Promise<void> {
    // console.log(`Invalidating feed cache for profile ${profileId}`);
    // Example: await redis.del(`feed:${profileId}`);
  }

  async warmupFeedCache(profileId: string): Promise<void> {
    // console.log(`Warming up feed cache for profile ${profileId}`);
    // Example: const feed = await this.getHomeFeed(profileId, { limit: 20 });
    // await redis.set(`feed:${profileId}`, JSON.stringify(feed), 'EX', 3600);
  }
}

---

§ 5. SRC/SERVER/SERVICES/FOLLOW-SERVICE.TS (150 RIGHE)
typescript
import { PrismaClient, Follow, FollowStatus, Profile } from '@prisma/client';
import { ListParams, PaginatedResult } from '~/lib/validations/social';
import { ProfileService } from './profile-service';
import { NotificationService } from './notification-service'; // Assuming a notification service

const prisma = new PrismaClient();
const profileService = new ProfileService();
const notificationService = new NotificationService();

export class FollowService {
  async follow(followerId: string, followingId: string): Promise<Follow> {
    if (followerId === followingId) throw new Error('Cannot follow yourself');

    const followingProfile = await profileService.getById(followingId);
    if (!followingProfile) throw new Error('Profile to follow not found');

    const status = followingProfile.isPrivate ? FollowStatus.PENDING : FollowStatus.ACCEPTED;

    try {
      const follow = await prisma.follow.create({
        data: {
          followerId,
          followingId,
          status,
        },
      });

      if (status === FollowStatus.ACCEPTED) {
        await profileService.incrementStat(followerId, 'following', 1);
        await profileService.incrementStat(followingId, 'followers', 1);
        await notificationService.create(followingId, followerId, 'FOLLOW');
      } else {
        // Real-time: notify the private account about a pending request
        await notificationService.create(followingId, followerId, 'FOLLOW_REQUEST'); // Custom notification type
      }
      return follow;
    } catch (error: any) {
      if (error.code === 'P2002') {
        // Already following or request already pending
        const existingFollow = await prisma.follow.findUnique({
          where: { followerId_followingId: { followerId, followingId } },
        });
        if (existingFollow) return existingFollow;
      }
      throw error;
    }
  }

  async unfollow(followerId: string, followingId: string): Promise<void> {
    const follow = await prisma.follow.delete({
      where: {
        followerId_followingId: { followerId, followingId },
      },
    });

    if (follow.status === FollowStatus.ACCEPTED) {
      await profileService.incrementStat(followerId, 'following', -1);
      await profileService.incrementStat(followingId, 'followers', -1);
    }
    // Real-time: invalidate feeds, remove notifications
  }

  async acceptFollowRequest(followId: string): Promise<Follow> {
    const follow = await prisma.follow.update({
      where: { id: followId, status: FollowStatus.PENDING },
      data: { status: FollowStatus.ACCEPTED },
    });
    await profileService.incrementStat(follow.followerId, 'following', 1);
    await profileService.incrementStat(follow.followingId, 'followers', 1);
    await notificationService.create(follow.followerId, follow.followingId, 'FOLLOW_ACCEPTED'); // Custom notification type
    return follow;
  }

  async rejectFollowRequest(followId: string): Promise<void> {
    await prisma.follow.delete({
      where: { id: followId, status: FollowStatus.PENDING },
    });
    // Real-time: notify requester of rejection
  }

  async isFollowing(followerId: string, followingId: string): Promise<boolean> {
    const follow = await prisma.follow.findUnique({
      where: { followerId_followingId: { followerId, followingId } },
    });
    return follow?.status === FollowStatus.ACCEPTED;
  }

  async getFollowStatus(followerId: string, followingId: string): Promise<FollowStatus | null> {
    const follow = await prisma.follow.findUnique({
      where: { followerId_followingId: { followerId, followingId } },
    });
    return follow?.status || null;
  }

  async getFollowers(profileId: string, params?: ListParams): Promise<PaginatedResult<Profile>> {
    const { limit = 10, cursor } = params || {};
    const followers = await prisma.follow.findMany({
      where: { followingId: profileId, status: FollowStatus.ACCEPTED },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { follower: true },
    });
    return {
      items: followers.map(f => f.follower),
      nextCursor: followers.length === limit ? followers[followers.length - 1]?.id : undefined,
    };
  }

  async getFollowing(profileId: string, params?: ListParams): Promise<PaginatedResult<Profile>> {
    const { limit = 10, cursor } = params || {};
    const following = await prisma.follow.findMany({
      where: { followerId: profileId, status: FollowStatus.ACCEPTED },
      take: limit,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { createdAt: 'desc' },
      include: { following: true },
    });
    return {
      items: following.map(f => f.following),
      nextCursor: following.length === limit ? following[following.length - 1]?.id : undefined,
    };
  }

  async getPendingRequests(profileId: string): Promise<Follow[]> {
    return prisma.follow.findMany({
      where: { followingId: profileId, status: FollowStatus.PENDING },
      include: { follower: true },
      orderBy: { createdAt: 'asc' },
    });
  }

  async getMutualFollowers(profileId1: string, profileId2: string): Promise<Profile[]> {
    const p1FollowingP2 = await prisma.follow.findMany({
      where: { followerId: profileId1, status: FollowStatus.ACCEPTED },
      select: { followingId: true },
    });
    const p1FollowingIds = p1FollowingP2.map(f => f.followingId);

    const mutualFollows = await prisma.follow.findMany({
      where: {
        followerId: profileId2,
        followingId: { in: p1FollowingIds },
        status: FollowStatus.ACCEPTED,
      },
      include: { following: true },
    });

    return mutualFollows.map(f => f.following);
  }
}

// Placeholder for NotificationService (defined in post-service.ts, but imported here)
class NotificationService {
  async create(recipientId: string, senderId: string, type: string, postId?: string, commentId?: string) {
    await prisma.notification.create({
      data: {
        recipientId,
        senderId,
        type: type as any,
        postId,
        commentId,
      },
    });
  }
}

---

§ 6. SRC/SERVER/TRPC/ROUTERS/SOCIAL.TS (250 RIGHE)
typescript
import { z } from 'zod';
import { publicProcedure, router, protectedProcedure } from '../trpc';
import { ProfileService } from '~/server/services/profile-service';
import { PostService } from '~/server/services/post-service';
import { FeedService } from '~/server/services/feed-service';
import { FollowService } from '~/server/services/follow-service';
import {
  createProfileSchema,
  updateProfileSchema,
  searchParamsSchema,
  listParamsSchema,
  createPostSchema,
  updatePostSchema,
  feedParamsSchema,
} from '~/lib/validations/social';

const profileService = new ProfileService();
const postService = new PostService();
const feedService = new FeedService();
const followService = new FollowService();

export const socialRouter = router({
  profile: router({
    getById: publicProcedure
      .input(z.object({ id: z.string() }))
      .query(({ input }) => profileService.getById(input.id)),
    getByUsername: publicProcedure
      .input(z.object({ username: z.string() }))
      .query(({ input }) => profileService.getByUsername(input.username)),
    getByUserId: protectedProcedure
      .input(z.object({ userId: z.string() }))
      .query(({ input }) => profileService.getByUserId(input.userId)),
    create: protectedProcedure
      .input(createProfileSchema)
      .mutation(({ ctx, input }) => profileService.create(ctx.session.user.id, input)),
    update: protectedProcedure
      .input(updateProfileSchema.extend({ profileId: z.string() }))
      .mutation(({ input }) => {
        const { profileId, ...data } = input;
        return profileService.update(profileId, data);
      }),
    search: publicProcedure
      .input(searchParamsSchema.extend({ query: z.string() }))
      .query(({ input }) => profileService.search(input.query, input)),
    isUsernameAvailable: publicProcedure
      .input(z.object({ username: z.string() }))
      .query(({ input }) => profileService.isUsernameAvailable(input.username)),
    getSuggested: protectedProcedure
      .input(z.object({ limit: z.number().optional() }))
      .query(({ ctx, input }) => profileService.getSuggested(ctx.session.user.profileId, input.limit)),
  }),

  post: router({
    getById: publicProcedure
      .input(z.object({ id: z.string() }))
      .query(async ({ input }) => {
        const post = await postService.getById(input.id);
        if (post) await postService.incrementViews(input.id); // Increment views on single post view
        return post;
      }),
    create: protectedProcedure
      .input(createPostSchema)
      .mutation(({ ctx, input }) => postService.create(ctx.session.user.profileId, input)),
    update: protectedProcedure
      .input(updatePostSchema.extend({ postId: z.string() }))
      .mutation(({ input }) => {
        const { postId, ...data } = input;
        return postService.update(postId, data);
      }),
    delete: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ input }) => postService.delete(input.postId)),
    getByAuthor: publicProcedure
      .input(listParamsSchema.extend({ authorId: z.string() }))
      .query(({ input }) => postService.getByAuthor(input.authorId, input)),
    like: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.like(input.postId, ctx.session.user.profileId)),
    unlike: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.unlike(input.postId, ctx.session.user.profileId)),
    isLiked: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .query(({ ctx, input }) => postService.isLiked(input.postId, ctx.session.user.profileId)),
    bookmark: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.bookmark(input.postId, ctx.session.user.profileId)),
    unbookmark: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.unbookmark(input.postId, ctx.session.user.profileId)),
    getBookmarks: protectedProcedure
      .input(listParamsSchema)
      .query(({ ctx, input }) => postService.getBookmarks(ctx.session.user.profileId, input)),
    repost: protectedProcedure
      .input(z.object({ postId: z.string(), quote: z.string().optional() }))
      .mutation(({ ctx, input }) => postService.repost(input.postId, ctx.session.user.profileId, input.quote)),
    unrepost: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.unrepost(input.postId, ctx.session.user.profileId)),
    pin: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.pin(input.postId, ctx.session.user.profileId)),
    unpin: protectedProcedure
      .input(z.object({ postId: z.string() }))
      .mutation(({ ctx, input }) => postService.unpin(input.postId, ctx.session.user.profileId)),
  }),

  feed: router({
    getHomeFeed: protectedProcedure
      .input(feedParamsSchema)
      .query(({ ctx, input }) => feedService.getHomeFeed(ctx.session.user.profileId, input)),
    getExploreFeed: protectedProcedure
      .input(feedParamsSchema)
      .query(({ ctx, input }) => feedService.getExploreFeed(ctx.session.user.profileId, input)),
    getProfileFeed: publicProcedure
      .input(feedParamsSchema.extend({ profileId: z.string(), viewerId: z.string().optional() }))
      .query(({ input }) => feedService.getProfileFeed(input.profileId, input.viewerId, input)),
  }),

  follow: router({
    follow: protectedProcedure
      .input(z.object({ followingId: z.string() }))
      .mutation(({ ctx, input }) => followService.follow(ctx.session.user.profileId, input.followingId)),
    unfollow: protectedProcedure
      .input(z.object({ followingId: z.string() }))
      .mutation(({ ctx, input }) => followService.unfollow(ctx.session.user.profileId, input.followingId)),
    acceptRequest: protectedProcedure
      .input(z.object({ followId: z.string() }))
      .mutation(({ input }) => followService.acceptFollowRequest(input.followId)),
    rejectRequest: protectedProcedure
      .input(z.object({ followId: z.string() }))
      .mutation(({ input }) => followService.rejectFollowRequest(input.followId)),
    isFollowing: protectedProcedure
      .input(z.object({ followingId: z.string() }))
      .query(({ ctx, input }) => followService.isFollowing(ctx.session.user.profileId, input.followingId)),
    getFollowStatus: protectedProcedure
      .input(z.object({ followingId: z.string() }))
      .query(({ ctx, input }) => followService.getFollowStatus(ctx.session.user.profileId, input.followingId)),
    getFollowers: publicProcedure
      .input(listParamsSchema.extend({ profileId: z.string() }))
      .query(({ input }) => followService.getFollowers(input.profileId, input)),
    getFollowing: publicProcedure
      .input(listParamsSchema.extend({ profileId: z.string() }))
      .query(({ input }) => followService.getFollowing(input.profileId, input)),
    getPendingRequests: protectedProcedure
      .query(({ ctx }) => followService.getPendingRequests(ctx.session.user.profileId)),
  }),
});

// Mock tRPC context and procedures for demonstration
// In a real app, these would be defined in your tRPC setup
type Session = { user: { id: string; profileId: string; email: string } };
const createContext = () => ({ session: { user: { id: 'user123', profileId: 'profile123', email: 'test@example.com' } } as Session });
const publicProcedure = publicProcedure; // Replace with actual publicProcedure
const protectedProcedure = protectedProcedure; // Replace with actual protectedProcedure

---

§ 7. SRC/LIB/VALIDATIONS/SOCIAL.TS (100 RIGHE)
typescript
import { z } from 'zod';
import { PostVisibility } from '@prisma/client';

// --- General Schemas ---
export const listParamsSchema = z.object({
  limit: z.number().min(1).max(100).default(10).optional(),
  cursor: z.string().optional(),
}).optional();

export type ListParams = z.infer<typeof listParamsSchema>;

export const searchParamsSchema = z.object({
  limit: z.number().min(1).max(100).default(10).optional(),
  offset: z.number().min(0).default(0).optional(),
}).optional();

export type SearchParams = z.infer<typeof searchParamsSchema>;

export const paginatedResultSchema = z.object({
  items: z.array(z.any()), // Type will be refined by specific usage
  nextCursor: z.string().optional().nullable(),
});

export type PaginatedResult<T> = {
  items: T[];
  nextCursor?: string | null;
};

// --- Profile Schemas ---
export const createProfileSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/, "Username can only contain letters, numbers, and underscores."),
  displayName: z.string().min(1).max(50),
  bio: z.string().max(250).optional().nullable(),
  avatar: z.string().url().optional().nullable(),
  coverImage: z.string().url().optional().nullable(),
  website: z.string().url().optional().nullable(),
  location: z.string().max(100).optional().nullable(),
  isPrivate: z.boolean().default(false).optional(),
});

export type CreateProfileInput = z.infer<typeof createProfileSchema>;

export const updateProfileSchema = createProfileSchema.partial();
export type UpdateProfileInput = z.infer<typeof updateProfileSchema>;

// --- Post Schemas ---
export const createPostSchema = z.object({
  content: z.string().min(1).max(500),
  images: z.array(z.string().url()).max(4).optional(),
  videos: z.array(z.string().url()).max(1).optional(),
  visibility: z.nativeEnum(PostVisibility).default(PostVisibility.PUBLIC).optional(),
  repostOfId: z.string().optional().nullable(),
});

export type CreatePostInput = z.infer<typeof createPostSchema>;

export const updatePostSchema = createPostSchema.partial();
export type UpdatePostInput = z.infer<typeof updatePostSchema>;

// --- Feed Schemas ---
export const feedParamsSchema = listParamsSchema.extend({
  // Add any specific feed parameters here, e.g., filters
  filterBy: z.enum(['all', 'following', 'popular']).default('all').optional(),
}).optional();

export type FeedParams = z.infer<typeof feedParamsSchema>;

---

§ 8. SRC/HOOKS/USE-PROFILE.TS (80 RIGHE)
typescript
import { trpc } from '~/utils/trpc';
import { CreateProfileInput, UpdateProfileInput } from '~/lib/validations/social';
import { Profile } from '@prisma/client';
import { useQueryClient } from '@tanstack/react-query';

export const useProfile = (username?: string, profileId?: string, userId?: string) => {
  const queryClient = useQueryClient();

  const { data: profile, isLoading: isLoadingProfile, error: profileError } = trpc.profile.getByUsername.useQuery(
    { username: username! },
    { enabled: !!username }
  );

  const { data: profileById, isLoading: isLoadingProfileById } = trpc.profile.getById.useQuery(
    { id: profileId! },
    { enabled: !!profileId }
  );

  const { data: profileByUserId, isLoading: isLoadingProfileByUserId } = trpc.profile.getByUserId.useQuery(
    { userId: userId! },
    { enabled: !!userId }
  );

  const createProfileMutation = trpc.profile.create.useMutation({
    onSuccess: (newProfile) => {
      queryClient.invalidateQueries(['profile', 'getByUserId', { userId: newProfile.userId }]);
      queryClient.invalidateQueries(['profile', 'getByUsername', { username: newProfile.username }]);
    },
  });

  const updateProfileMutation = trpc.profile.update.useMutation({
    onMutate: async (updatedData) => {
      // Optimistic update
      await queryClient.cancelQueries(['profile', 'getByUsername', { username }]);
      const previousProfile = queryClient.getQueryData(['profile', 'getByUsername', { username }]);
      queryClient.setQueryData(['profile', 'getByUsername', { username }], (old: Profile | undefined) =>
        old ? { ...old, ...updatedData } : old
      );
      return { previousProfile };
    },
    onError: (err, newProfile, context) => {
      // Rollback on error
      queryClient.setQueryData(['profile', 'getByUsername', { username }], context?.previousProfile);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['profile', 'getByUsername', { username }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: profile?.id }]);
    },
  });

  const { data: isUsernameAvailable, isLoading: isLoadingUsernameAvailability } = trpc.profile.isUsernameAvailable.useQuery(
    { username: username! },
    { enabled: !!username && username.length > 0, staleTime: 5 * 60 * 1000 } // Cache for 5 minutes
  );

  const searchProfilesQuery = trpc.profile.search.useQuery;
  const getSuggestedProfilesQuery = trpc.profile.getSuggested.useQuery;

  return {
    profile: profile || profileById || profileByUserId,
    isLoadingProfile: isLoadingProfile || isLoadingProfileById || isLoadingProfileByUserId,
    profileError,
    createProfile: (data: CreateProfileInput) => createProfileMutation.mutate(data),
    updateProfile: (profileId: string, data: UpdateProfileInput) => updateProfileMutation.mutate({ profileId, ...data }),
    isUsernameAvailable,
    isLoadingUsernameAvailability,
    searchProfilesQuery,
    getSuggestedProfilesQuery,
  };
};

---

§ 9. SRC/HOOKS/USE-POSTS.TS (100 RIGHE)
typescript
import { trpc } from '~/utils/trpc';
import { CreatePostInput, UpdatePostInput } from '~/lib/validations/social';
import { Post } from '@prisma/client';
import { useQueryClient } from '@tanstack/react-query';

export const usePosts = () => {
  const queryClient = useQueryClient();

  const getPostByIdQuery = (postId: string) => trpc.post.getById.useQuery({ id: postId }, { enabled: !!postId });
  const getPostsByAuthorQuery = (authorId: string) => trpc.post.getByAuthor.useInfiniteQuery(
    { authorId, limit: 10 },
    { getNextPageParam: (lastPage) => lastPage.nextCursor }
  );
  const getBookmarksQuery = (profileId: string) => trpc.post.getBookmarks.useInfiniteQuery(
    { profileId, limit: 10 },
    { getNextPageParam: (lastPage) => lastPage.nextCursor }
  );

  const createPostMutation = trpc.post.create.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
      queryClient.invalidateQueries(['post', 'getByAuthor']);
    },
  });

  const updatePostMutation = trpc.post.update.useMutation({
    onMutate: async (updatedPost) => {
      await queryClient.cancelQueries(['post', 'getById', { id: updatedPost.postId }]);
      const previousPost = queryClient.getQueryData(['post', 'getById', { id: updatedPost.postId }]);
      queryClient.setQueryData(['post', 'getById', { id: updatedPost.postId }], (old: Post | undefined) =>
        old ? { ...old, ...updatedPost } : old
      );
      return { previousPost };
    },
    onError: (err, newPost, context) => {
      queryClient.setQueryData(['post', 'getById', { id: newPost.postId }], context?.previousPost);
    },
    onSettled: (data) => {
      queryClient.invalidateQueries(['post', 'getById', { id: data?.id }]);
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
      queryClient.invalidateQueries(['post', 'getByAuthor']);
    },
  });

  const deletePostMutation = trpc.post.delete.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
      queryClient.invalidateQueries(['post', 'getByAuthor']);
    },
  });

  const likePostMutation = trpc.post.like.useMutation({
    onMutate: async ({ postId }) => {
      await queryClient.cancelQueries(['post', 'getById', { id: postId }]);
      const previousPost = queryClient.getQueryData(['post', 'getById', { id: postId }]);
      queryClient.setQueryData(['post', 'getById', { id: postId }], (old: Post | undefined) =>
        old ? { ...old, likesCount: old.likesCount + 1 } : old
      );
      return { previousPost };
    },
    onError: (err, { postId }, context) => {
      queryClient.setQueryData(['post', 'getById', { id: postId }], context?.previousPost);
    },
    onSettled: (data, error, { postId }) => {
      queryClient.invalidateQueries(['post', 'getById', { id: postId }]);
      queryClient.invalidateQueries(['feed', 'getHomeFeed']); // Potentially update feed
    },
  });

  const unlikePostMutation = trpc.post.unlike.useMutation({
    onMutate: async ({ postId }) => {
      await queryClient.cancelQueries(['post', 'getById', { id: postId }]);
      const previousPost = queryClient.getQueryData(['post', 'getById', { id: postId }]);
      queryClient.setQueryData(['post', 'getById', { id: postId }], (old: Post | undefined) =>
        old ? { ...old, likesCount: old.likesCount - 1 } : old
      );
      return { previousPost };
    },
    onError: (err, { postId }, context) => {
      queryClient.setQueryData(['post', 'getById', { id: postId }], context?.previousPost);
    },
    onSettled: (data, error, { postId }) => {
      queryClient.invalidateQueries(['post', 'getById', { id: postId }]);
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
    },
  });

  const isLikedQuery = (postId: string) => trpc.post.isLiked.useQuery({ postId }, { enabled: !!postId });

  const bookmarkPostMutation = trpc.post.bookmark.useMutation({
    onSuccess: () => queryClient.invalidateQueries(['post', 'getBookmarks']),
  });
  const unbookmarkPostMutation = trpc.post.unbookmark.useMutation({
    onSuccess: () => queryClient.invalidateQueries(['post', 'getBookmarks']),
  });

  const repostMutation = trpc.post.repost.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
      queryClient.invalidateQueries(['post', 'getByAuthor']);
    },
  });
  const unrepostMutation = trpc.post.unrepost.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['feed', 'getHomeFeed']);
      queryClient.invalidateQueries(['post', 'getByAuthor']);
    },
  });

  const pinPostMutation = trpc.post.pin.useMutation({
    onSuccess: () => queryClient.invalidateQueries(['post', 'getByAuthor']),
  });
  const unpinPostMutation = trpc.post.unpin.useMutation({
    onSuccess: () => queryClient.invalidateQueries(['post', 'getByAuthor']),
  });

  return {
    getPostByIdQuery,
    getPostsByAuthorQuery,
    getBookmarksQuery,
    createPost: (data: CreatePostInput) => createPostMutation.mutate(data),
    updatePost: (postId: string, data: UpdatePostInput) => updatePostMutation.mutate({ postId, ...data }),
    deletePost: (postId: string) => deletePostMutation.mutate({ postId }),
    likePost: (postId: string) => likePostMutation.mutate({ postId }),
    unlikePost: (postId: string) => unlikePostMutation.mutate({ postId }),
    isLikedQuery,
    bookmarkPost: (postId: string) => bookmarkPostMutation.mutate({ postId }),
    unbookmarkPost: (postId: string) => unbookmarkPostMutation.mutate({ postId }),
    repost: (postId: string, quote?: string) => repostMutation.mutate({ postId, quote }),
    unrepost: (postId: string) => unrepostMutation.mutate({ postId }),
    pinPost: (postId: string) => pinPostMutation.mutate({ postId }),
    unpinPost: (postId: string) => unpinPostMutation.mutate({ postId }),
  };
};

---

§ 10. SRC/HOOKS/USE-FEED.TS (80 RIGHE)
typescript
import { trpc } from '~/utils/trpc';
import { FeedParams } from '~/lib/validations/social';

export const useFeed = () => {
  const useHomeFeed = (params?: FeedParams) => trpc.feed.getHomeFeed.useInfiniteQuery(
    { ...params, limit: 10 },
    { getNextPageParam: (lastPage) => lastPage.nextCursor }
  );

  const useExploreFeed = (params?: FeedParams) => trpc.feed.getExploreFeed.useInfiniteQuery(
    { ...params, limit: 10 },
    { getNextPageParam: (lastPage) => lastPage.nextCursor }
  );

  const useProfileFeed = (profileId: string, viewerId?: string, params?: FeedParams) => trpc.feed.getProfileFeed.useInfiniteQuery(
    { profileId, viewerId, ...params, limit: 10 },
    { getNextPageParam: (lastPage) => lastPage.nextCursor, enabled: !!profileId }
  );

  return {
    useHomeFeed,
    useExploreFeed,
    useProfileFeed,
  };
};

---

§ 11. SRC/HOOKS/USE-FOLLOW.TS (60 RIGHE)
typescript
import { trpc } from '~/utils/trpc';
import { useQueryClient } from '@tanstack/react-query';
import { FollowStatus } from '@prisma/client';

export const useFollow = (followerId: string, followingId: string) => {
  const queryClient = useQueryClient();

  const { data: followStatus, isLoading: isLoadingFollowStatus } = trpc.follow.getFollowStatus.useQuery(
    { followingId },
    { enabled: !!followerId && !!followingId }
  );

  const followMutation = trpc.follow.follow.useMutation({
    onMutate: async () => {
      await queryClient.cancelQueries(['follow', 'getFollowStatus', { followingId }]);
      const previousStatus = queryClient.getQueryData(['follow', 'getFollowStatus', { followingId }]);
      queryClient.setQueryData(['follow', 'getFollowStatus', { followingId }], (old: FollowStatus | undefined) => {
        // Optimistically set to PENDING or ACCEPTED
        return old === FollowStatus.PENDING ? FollowStatus.PENDING : FollowStatus.ACCEPTED;
      });
      return { previousStatus };
    },
    onError: (err, newFollow, context) => {
      queryClient.setQueryData(['follow', 'getFollowStatus', { followingId }], context?.previousStatus);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['follow', 'getFollowStatus', { followingId }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: followingId }]); // Invalidate target profile stats
      queryClient.invalidateQueries(['profile', 'getById', { id: followerId }]); // Invalidate current user profile stats
    },
  });

  const unfollowMutation = trpc.follow.unfollow.useMutation({
    onMutate: async () => {
      await queryClient.cancelQueries(['follow', 'getFollowStatus', { followingId }]);
      const previousStatus = queryClient.getQueryData(['follow', 'getFollowStatus', { followingId }]);
      queryClient.setQueryData(['follow', 'getFollowStatus', { followingId }], () => null); // Optimistically unfollow
      return { previousStatus };
    },
    onError: (err, newFollow, context) => {
      queryClient.setQueryData(['follow', 'getFollowStatus', { followingId }], context?.previousStatus);
    },
    onSettled: () => {
      queryClient.invalidateQueries(['follow', 'getFollowStatus', { followingId }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: followingId }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: followerId }]);
    },
  });

  const acceptFollowRequestMutation = trpc.follow.acceptRequest.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['follow', 'getPendingRequests']);
      queryClient.invalidateQueries(['follow', 'getFollowers', { profileId: followerId }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: followerId }]);
      queryClient.invalidateQueries(['profile', 'getById', { id: followingId }]);
    },
  });

  const rejectFollowRequestMutation = trpc.follow.rejectRequest.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(['follow', 'getPendingRequests']);
    },
  });

  return {
    followStatus,
    isLoadingFollowStatus,
    follow: () => followMutation.mutate({ followingId }),
    unfollow: () => unfollowMutation.mutate({ followingId }),
    acceptFollowRequest: (followId: string) => acceptFollowRequestMutation.mutate({ followId }),
    rejectFollowRequest: (followId: string) => rejectFollowRequestMutation.mutate({ followId }),
  };
};

---

§ 12. SRC/COMPONENTS/SOCIAL/PROFILE-CARD.TSX (100 RIGHE)
typescript
import React from 'react';
import Link from 'next/link';
import Image from 'next/image';
import { Profile } from '@prisma/client';
import { FollowButton } from './follow-button';
import { useSession } from 'next-auth/react';

interface ProfileCardProps {
  profile: Profile;
  showFollowButton?: boolean;
}

export const ProfileCard: React.FC<ProfileCardProps> = ({ profile, showFollowButton = true }) => {
  const { data: session } = useSession();
  const currentProfileId = session?.user?.profileId;

  const isOwnProfile = currentProfileId === profile.id;

  return (
    <div className="flex items-center justify-between p-4 border-b border-gray-200 dark:border-gray-700">
      <Link href={`/${profile.username}`} className="flex items-center space-x-3 group">
        <div className="relative w-12 h-12 rounded-full overflow-hidden flex-shrink-0">
          <Image
            src={profile.avatar || '/default-avatar.png'}
            alt={`${profile.displayName}'s avatar`}
            fill
            className="object-cover group-hover:brightness-90 transition-all"
            sizes="48px"
            priority
          />
        </div>
        <div className="flex flex-col">
          <span className="font-bold text-lg group-hover:text-blue-500 transition-colors">
            {profile.displayName}
            {profile.isVerified && (
              <span className="ml-1 text-blue-500 text-sm" title="Verified Account">
                ✓
              </span>
            )}
          </span>
          <span className="text-gray-500 dark:text-gray-400 text-sm">@{profile.username}</span>
          {profile.bio && (
            <p className="text-gray-700 dark:text-gray-300 text-sm mt-1 line-clamp-2">
              {profile.bio}
            </p>
          )}
          <div className="flex items-center space-x-3 text-gray-500 dark:text-gray-400 text-sm mt-2">
            <span>
              <span className="font-semibold text-gray-800 dark:text-gray-200">{profile.followersCount}</span> Followers
            </span>
            <span>
              <span className="font-semibold text-gray-800 dark:text-gray-200">{profile.followingCount}</span> Following
            </span>
          </div>
        </div>
      </Link>
      {showFollowButton && !isOwnProfile && currentProfileId && (
        <FollowButton followerId={currentProfileId} followingId={profile.id} />
      )}
    </div>
  );
};

---

§ 13. SRC/COMPONENTS/SOCIAL/PROFILE-HEADER.TSX (150 RIGHE)
typescript
import React from 'react';
import Image from 'next/image';
import Link from 'next/link';
import { Profile } from '@prisma/client';
import { FollowButton } from './follow-button';
import { useSession } from 'next-auth/react';
import { Button } from '../ui/button'; // Assuming a UI button component
import { CalendarIcon, LinkIcon, MapPinIcon } from 'lucide-react'; // Assuming lucide-react icons
import { format } from 'date-fns';

interface ProfileHeaderProps {
  profile: Profile;
  // Add an 'onEdit' prop if you want an edit button
  onEdit?: () => void;
}

export const ProfileHeader: React.FC<ProfileHeaderProps> = ({ profile, onEdit }) => {
  const { data: session } = useSession();
  const currentProfileId = session?.user?.profileId;

  const isOwnProfile = currentProfileId === profile.id;

  if (!profile) {
    return <div className="p-4 text-center text-gray-500">Profile not found.</div>;
  }

  return (
    <div className="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700">
      {/* Cover Image */}
      <div className="relative h-48 bg-gray-300 dark:bg-gray-700">
        {profile.coverImage && (
          <Image
            src={profile.coverImage}
            alt={`${profile.displayName}'s cover`}
            fill
            className="object-cover"
            sizes="100vw"
            priority
          />
        )}
      </div>

      <div className="p-4">
        <div className="flex justify-between items-end -mt-20 sm:-mt-16">
          {/* Avatar */}
          <div className="relative w-32 h-32 sm:w-40 sm:h-40 rounded-full border-4 border-white dark:border-gray-800 overflow-hidden bg-gray-200 dark:bg-gray-600">
            <Image
              src={profile.avatar || '/default-avatar.png'}
              alt={`${profile.displayName}'s avatar`}
              fill
              className="object-cover"
              sizes="(max-width: 768px) 128px, 160px"
              priority
            />
          </div>

          {/* Actions */}
          <div className="flex space-x-2">
            {isOwnProfile ? (
              onEdit && (
                <Button variant="outline" onClick={onEdit}>
                  Edit Profile
                </Button>
              )
            ) : (
              currentProfileId && <FollowButton followerId={currentProfileId} followingId={profile.id} />
            )}
          </div>
        </div>

        {/* Profile Info */}
        <div className="mt-4">
          <h1 className="text-2xl font-bold flex items-center">
            {profile.displayName}
            {profile.isVerified && (
              <span className="ml-2 text-blue-500 text-xl" title="Verified Account">
                ✓
              </span>
            )}
          </h1>
          <p className="text-gray-500 dark:text-gray-400 text-sm">@{profile.username}</p>
          {profile.bio && <p className="mt-2 text-gray-800 dark:text-gray-200">{profile.bio}</p>}

          <div className="mt-3 flex flex-wrap items-center text-gray-500 dark:text-gray-400 text-sm space-x-4">
            {profile.location && (
              <span className="flex items-center">
                <MapPinIcon className="w-4 h-4 mr-1" /> {profile.location}
              </span>
            )}
            {profile.website && (
              <Link href={profile.website} target="_blank" rel="noopener noreferrer" className="flex items-center hover:underline text-blue-500">
                <LinkIcon className="w-4 h-4 mr-1" /> {new URL(profile.website).hostname}
              </Link>
            )}
            <span className="flex items-center">
              <CalendarIcon className="w-4 h-4 mr-1" /> Joined {format(new Date(profile.createdAt), 'MMM yyyy')}
            </span>
          </div>

          {/* Stats */}
          <div className="mt-4 flex space-x-6 text-gray-800 dark:text-gray-200">
            <Link href={`/${profile.username}/following`} className="hover:underline">
              <span className="font-bold">{profile.followingCount}</span>{' '}
              <span className="text-gray-500 dark:text-gray-400">Following</span>
            </Link>
            <Link href={`/${profile.username}/followers`} className="hover:underline">
              <span className="font-bold">{profile.followersCount}</span>{' '}
              <span className="text-gray-500 dark:text-gray-400">Followers</span>
            </Link>
          </div>
        </div>
      </div>
    </div>
  );
};

---

§ 14. SRC/COMPONENTS/SOCIAL/POST-CARD.TSX (200 RIGHE)
typescript
import React from 'react';
import Link from 'next/link';
import Image from 'next/image';
import { Post, Profile } from '@prisma/client';
import { formatDistanceToNowStrict } from 'date-fns';
import {
  HeartIcon, MessageCircleIcon, Repeat2Icon, BookmarkIcon, Share2Icon, MoreHorizontalIcon
} from 'lucide-react';
import { Button } from '../ui/button'; // Assuming a UI button component
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger
} from '../ui/dropdown-menu'; // Assuming a UI dropdown menu
import { usePosts } from '~/hooks/use-posts';
import { useSession } from 'next-auth/react';
import { cn } from '~/lib/utils'; // Utility for conditional classnames

interface PostCardProps {
  post: Post & { author: Profile; repostOf?: (Post & { author: Profile }) | null };
  viewerProfileId?: string;
}

const highlightText = (text: string) => {
  // Highlight mentions (@username) and hashtags (#tag)
  return text.split(/(\s)/).map((segment, index) => {
    if (segment.startsWith('@')) {
      const username = segment.substring(1);
      return (
        <Link key={index} href={`/${username}`} className="text-blue-500 hover:underline">
          {segment}
        </Link>
      );
    }
    if (segment.startsWith('#')) {
      const hashtag = segment.substring(1);
      return (
        <Link key={index} href={`/explore?tag=${hashtag}`} className="text-blue-500 hover:underline">
          {segment}
        </Link>
      );
    }
    return segment;
  });
};

export const PostCard: React.FC<PostCardProps> = ({ post, viewerProfileId }) => {
  const { data: session } = useSession();
  const currentProfileId = session?.user?.profileId;
  const { likePost, unlikePost, isLikedQuery, bookmarkPost, unbookmarkPost, repost, deletePost } = usePosts();
  const { data: isLiked } = isLikedQuery(post.id);

  const isBookmarkedQuery = trpc.post.getBookmarks.useQuery({ profileId: currentProfileId! }, { enabled: !!currentProfileId });
  const isBookmarked = isBookmarkedQuery.data?.pages?.[0]?.items.some(b => b.id === post.id); // Simplified check

  const handleLike = () => {
    if (!currentProfileId) return;
    isLiked ? unlikePost(post.id) : likePost(post.id);
  };

  const handleBookmark = () => {
    if (!currentProfileId) return;
    isBookmarked ? unbookmarkPost(post.id) : bookmarkPost(post.id);
  };

  const handleRepost = () => {
    if (!currentProfileId) return;
    // For simplicity, direct repost without quote for now
    repost(post.id);
  };

  const handleDelete = () => {
    if (confirm('Are you sure you want to delete this post?')) {
      deletePost(post.id);
    }
  };

  const isAuthor = currentProfileId === post.authorId;

  return (
    <div className="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 p-4">
      {post.repostOf && (
        <div className="flex items-center text-gray-500 dark:text-gray-400 text-sm mb-2">
          <Repeat2Icon className="w-4 h-4 mr-1" />
          <Link href={`/${post.author.username}`} className="font-semibold hover:underline">
            @{post.author.username}
          </Link>{' '}
          reposted
        </div>
      )}

      <div className="flex items-start space-x-3">
        {/* Author Info */}
        <Link href={`/${post.repostOf ? post.repostOf.author.username : post.author.username}`}>
          <div className="relative w-10 h-10 rounded-full overflow-hidden flex-shrink-0">
            <Image
              src={post.repostOf ? (post.repostOf.author.avatar || '/default-avatar.png') : (post.author.avatar || '/default-avatar.png')}
              alt={`${post.author.displayName}'s avatar`}
              fill
              className="object-cover"
              sizes="40px"
            />
          </div>
        </Link>

        <div className="flex-1">
          <div className="flex justify-between items-center">
            <div className="flex items-center space-x-1">
              <Link href={`/${post.repostOf ? post.repostOf.author.username : post.author.username}`} className="font-bold hover:underline">
                {post.repostOf ? post.repostOf.author.displayName : post.author.displayName}
              </Link>
              <span className="text-gray-500 dark:text-gray-400">
                @{post.repostOf ? post.repostOf.author.username : post.author.username} ·{' '}
                {formatDistanceToNowStrict(new Date(post.createdAt), { addSuffix: true })}
              </span>
            </div>
            {isAuthor && (
              <DropdownMenu>
                <DropdownMenuTrigger asChild>
                  <Button variant="ghost" size="icon" className="rounded-full">
                    <MoreHorizontalIcon className="w-5 h-5 text-gray-500" />
                  </Button>
                </DropdownMenuTrigger>
                <DropdownMenuContent align="end">
                  <DropdownMenuItem onClick={handleDelete} className="text-red-500">
                    Delete Post
                  </DropdownMenuItem>
                  {/* Add other options like Edit, Pin, etc. */}
                </DropdownMenuContent>
              </DropdownMenu>
            )}
          </div>

          {/* Post Content */}
          <Link href={`/post/${post.id}`} className="block mt-1">
            <p className="text-gray-800 dark:text-gray-200 whitespace-pre-wrap">
              {highlightText(post.content)}
            </p>
          </Link>

          {/* Media Gallery */}
          {(post.images.length > 0 || post.videos.length > 0) && (
            <div className="mt-3 grid grid-cols-1 gap-2 rounded-lg overflow-hidden">
              {post.images.map((img, i) => (
                <div key={i} className="relative w-full h-60 bg-gray-100 dark:bg-gray-700">
                  <Image src={img} alt={`Post image ${i + 1}`} fill className="object-cover" sizes="(max-width: 768px) 100vw, 50vw" />
                </div>
              ))}
              {/* Videos would require a video player component */}
            </div>
          )}

          {/* Original Post if it's a Repost with Quote */}
          {post.repostOf && (
            <div className="mt-3 border border-gray-200 dark:border-gray-700 rounded-lg p-3">
              <PostCard post={post.repostOf} viewerProfileId={viewerProfileId} />
            </div>
          )}

          {/* Engagement Buttons */}
          <div className="flex justify-between mt-4 text-gray-500 dark:text-gray-400">
            <Button variant="ghost" size="sm" className={cn("flex items-center space-x-1 group", isLiked && "text-red-500")} onClick={handleLike}>
              <HeartIcon className={cn("w-5 h-5 group-hover:text-red-500", isLiked && "fill-current")} />
              <span>{post.likesCount}</span>
            </Button>
            <Button variant="ghost" size="sm" className="flex items-center space-x-1 group">
              <MessageCircleIcon className="w-5 h-5 group-hover:text-blue-500" />
              <span>{post.commentsCount}</span>
            </Button>
            <Button variant="ghost" size="sm" className="flex items-center space-x-1 group" onClick={handleRepost}>
              <Repeat2Icon className="w-5 h-5 group-hover:text-green-500" />
              <span>{post.sharesCount}</span>
            </Button>
            <Button variant="ghost" size="sm" className={cn("flex items-center space-x-1 group", isBookmarked && "text-yellow-500")} onClick={handleBookmark}>
              <BookmarkIcon className={cn("w-5 h-5 group-hover:text-yellow-500", isBookmarked && "fill-current")} />
            </Button>
            <Button variant="ghost" size="sm" className="flex items-center space-x-1 group">
              <Share2Icon className="w-5 h-5 group-hover:text-blue-500" />
            </Button>
          </div>
        </div>
      </div>
    </div>
  );
};

---

§ 15. SRC/COMPONENTS/SOCIAL/POST-COMPOSER.TSX (200 RIGHE)
typescript
import React, { useState, useRef, ChangeEvent, FormEvent } from 'react';
import Image from 'next/image';
import { useSession } from 'next-auth/react';
import { Button } from '../ui/button';
import { Textarea } from '../ui/textarea'; // Assuming a UI textarea component
import {
  ImageIcon, VideoIcon, SmileIcon, HashIcon, AtSignIcon, GlobeIcon, UsersIcon, LockIcon
} from 'lucide-react';
import {
  Select, SelectContent, SelectItem, SelectTrigger, SelectValue
} from '../ui/select'; // Assuming a UI select component
import { PostVisibility } from '@prisma/client';
import { usePosts } from '~/hooks/use-posts';
import { useProfile } from '~/hooks/use-profile';
import { CreatePostInput } from '~/lib/validations/social';

interface PostComposerProps {
  onPostSuccess?: () => void;
  initialContent?: string;
  repostOfId?: string;
}

export const PostComposer: React.FC<PostComposerProps> = ({ onPostSuccess, initialContent = '', repostOfId }) => {
  const { data: session } = useSession();
  const { profile: currentProfile } = useProfile(undefined, session?.user?.profileId);
  const { createPost } = usePosts();

  const [content, setContent] = useState(initialContent);
  const [images, setImages] = useState<string[]>([]); // Placeholder for image URLs
  const [videos, setVideos] = useState<string[]>([]); // Placeholder for video URLs
  const [visibility, setVisibility] = useState<PostVisibility>(PostVisibility.PUBLIC);
  const textareaRef = useRef<HTMLTextAreaElement>(null);

  const MAX_CHARS = 500;
  const charsRemaining = MAX_CHARS - content.length;

  const handleContentChange = (e: ChangeEvent<HTMLTextAreaElement>) => {
    setContent(e.target.value);
    // Auto-resize textarea
    if (textareaRef.current) {
      textareaRef.current.style.height = 'auto';
      textareaRef.current.style.height = textareaRef.current.scrollHeight + 'px';
    }
  };

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    if (!content.trim() && images.length === 0 && videos.length === 0) return;

    const postData: CreatePostInput = {
      content,
      images: images.length > 0 ? images : undefined,
      videos: videos.length > 0 ? videos : undefined,
      visibility,
      repostOfId,
    };

    try {
      await createPost(postData);
      setContent('');
      setImages([]);
      setVideos([]);
      setVisibility(PostVisibility.PUBLIC);
      if (textareaRef.current) {
        textareaRef.current.style.height = 'auto'; // Reset height
      }
      onPostSuccess?.();
    } catch (error) {
      console.error('Failed to create post:', error);
      // Handle error, show toast, etc.
    }
  };

  if (!currentProfile) {
    return <div className="p-4 text-center text-gray-500">Please create a profile to post.</div>;
  }

  return (
    <div className="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 p-4">
      <div className="flex space-x-3">
        <div className="relative w-10 h-10 rounded-full overflow-hidden flex-shrink-0">
          <Image
            src={currentProfile.avatar || '/default-avatar.png'}
            alt={`${currentProfile.displayName}'s avatar`}
            fill
            className="object-cover"
            sizes="40px"
          />
        </div>
        <form onSubmit={handleSubmit} className="flex-1">
          <Textarea
            ref={textareaRef}
            value={content}
            onChange={handleContentChange}
            placeholder={repostOfId ? "Add a comment or just repost..." : "What's happening?"}
            className="w-full resize-none border-none focus:ring-0 text-lg p-0 dark:bg-gray-800 dark:text-gray-200"
            rows={1}
            maxLength={MAX_CHARS}
          />

          {/* Media Preview (Placeholder) */}
          {images.length > 0 && (
            <div className="mt-3 grid grid-cols-2 gap-2">
              {images.map((img, i) => (
                <div key={i} className="relative w-full h-32 rounded-lg overflow-hidden">
                  <Image src={img} alt="Uploaded media" fill className="object-cover" />
                  {/* Add remove button */}
                </div>
              ))}
            </div>
          )}

          <div className="flex items-center justify-between mt-4 border-t border-gray-200 dark:border-gray-700 pt-3">
            <div className="flex space-x-2 text-blue-500">
              <Button type="button" variant="ghost" size="icon" title="Add image">
                <ImageIcon className="w-5 h-5" />
              </Button>
              <Button type="button" variant="ghost" size="icon" title="Add video">
                <VideoIcon className="w-5 h-5" />
              </Button>
              <Button type="button" variant="ghost" size="icon" title="Add emoji">
                <SmileIcon className="w-5 h-5" />
              </Button>
              <Button type="button" variant="ghost" size="icon" title="Mention user">
                <AtSignIcon className="w-5 h-5" />
              </Button>
              <Button type="button" variant="ghost" size="icon" title="Add hashtag">
                <HashIcon className="w-5 h-5" />
              </Button>
            </div>

            <div className="flex items-center space-x-4">
              <Select value={visibility} onValueChange={(value: PostVisibility) => setVisibility(value)}>
                <SelectTrigger className="w-[180px]">
                  <SelectValue placeholder="Who can see this?" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value={PostVisibility.PUBLIC}>
                    <div className="flex items-center"><GlobeIcon className="w-4 h-4 mr-2" /> Everyone</div>
                  </SelectItem>
                  <SelectItem value={PostVisibility.FOLLOWERS_ONLY}>
                    <div className="flex items-center"><UsersIcon className="w-4 h-4 mr-2" /> Followers only</div>
                  </SelectItem>
                  <SelectItem value={PostVisibility.PRIVATE}>
                    <div className="flex items-center"><LockIcon className="w-4 h-4 mr-2" /> Private</div>
                  </SelectItem>
                </SelectContent>
              </Select>

              <span className={`text-sm ${charsRemaining < 20 ? 'text-red-500' : 'text-gray-500 dark:text-gray-400'}`}>
                {charsRemaining}
              </span>
              <Button type="submit" disabled={!content.trim() && images.length === 0 && videos.length === 0}>
                {repostOfId ? "Repost" : "Post"}
              </Button>
            </div>
          </div>
        </form>
      </div>
    </div>
  );
};

---

§ 16. SRC/COMPONENTS/SOCIAL/FEED.TSX (100 RIGHE)
typescript
import React, { useEffect, useRef, useCallback } from 'react';
import { Post } from '@prisma/client';
import { PostCard } from './post-card';
import { InfiniteQueryHookResult } from '~/types/trpc'; // Custom type for infinite query hook
import { Loader2Icon } from 'lucide-react'; // Assuming lucide-react icons

interface FeedProps {
  queryHook: InfiniteQueryHookResult<Post>;
  viewerProfileId?: string;
  emptyMessage?: string;
}

export const Feed: React.FC<FeedProps> = ({ queryHook, viewerProfileId, emptyMessage = "No posts to display." }) => {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isLoading, isError, error } = queryHook;

  const observerTarget = useRef<HTMLDivElement>(null);

  const handleObserver = useCallback((entries: IntersectionObserverEntry[]) => {
    const target = entries[0];
    if (target.isIntersecting && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [fetchNextPage, hasNextPage, isFetchingNextPage]);

  useEffect(() => {
    const observer = new IntersectionObserver(handleObserver, {
      root: null,
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
  }, [handleObserver, observerTarget]);

  if (isLoading) {
    return (
      <div className="flex justify-center items-center p-8">
        <Loader2Icon className="h-8 w-8 animate-spin text-blue-500" />
      </div>
    );
  }

  if (isError) {
    return <div className="p-4 text-red-500">Error loading feed: {error?.message}</div>;
  }

  const allPosts = data?.pages.flatMap((page) => page.items) || [];

  if (allPosts.length === 0) {
    return (
      <div className="p-8 text-center text-gray-500 dark:text-gray-400">
        {emptyMessage}
      </div>
    );
  }

  return (
    <div>
      {allPosts.map((post) => (
        <PostCard key={post.id} post={post} viewerProfileId={viewerProfileId} />
      ))}
      {hasNextPage && (
        <div ref={observerTarget} className="flex justify-center items-center p-4">
          {isFetchingNextPage ? (
            <Loader2Icon className="h-6 w-6 animate-spin text-blue-500" />
          ) : (
            <button
              onClick={() => fetchNextPage()}
              disabled={!hasNextPage || isFetchingNextPage}
              className="text-blue-500 hover:underline"
            >
              Load more
            </button>
          )}
        </div>
      )}
      {!hasNextPage && allPosts.length > 0 && (
        <div className="p-4 text-center text-gray-500 dark:text-gray-400">
          You've reached the end of the feed.
        </div>
      )}
    </div>
  );
};

// Define InfiniteQueryHookResult type (adjust based on your actual tRPC setup)
type InfiniteQueryHookResult<T> = {
  data: { pages: { items: T[]; nextCursor?: string | null }[] } | undefined;
  fetchNextPage: () => Promise<any>;
  hasNextPage: boolean | undefined;
  isFetchingNextPage: boolean;
  isLoading: boolean;
  isError: boolean;
  error: any;
};

---

§ 17. SRC/COMPONENTS/SOCIAL/FOLLOW-BUTTON.TSX (80 RIGHE)
typescript
import React from 'react';
import { Button } from '../ui/button'; // Assuming a UI button component
import { useFollow } from '~/hooks/use-follow';
import { FollowStatus } from '@prisma/client';
import { Loader2Icon } from 'lucide-react';

interface FollowButtonProps {
  followerId: string; // The ID of the current user (who is initiating the follow/unfollow)
  followingId: string; // The ID of the profile being followed/unfollowed
}

export const FollowButton: React.FC<FollowButtonProps> = ({ followerId, followingId }) => {
  const { followStatus, isLoadingFollowStatus, follow, unfollow, acceptFollowRequest, rejectFollowRequest } = useFollow(followerId, followingId);

  if (isLoadingFollowStatus) {
    return (
      <Button disabled>
        <Loader2Icon className="mr-2 h-4 w-4 animate-spin" /> Loading...
      </Button>
    );
  }

  const handleFollowClick = () => {
    if (followStatus === FollowStatus.ACCEPTED || followStatus === FollowStatus.PENDING) {
      unfollow();
    } else {
      follow();
    }
  };

  const buttonText = () => {
    switch (followStatus) {
      case FollowStatus.ACCEPTED:
        return 'Following';
      case FollowStatus.PENDING:
        return 'Requested';
      case FollowStatus.REJECTED: // Should not happen often, but handle it
      default:
        return 'Follow';
    }
  };

  const buttonVariant = () => {
    switch (followStatus) {
      case FollowStatus.ACCEPTED:
        return 'outline';
      case FollowStatus.PENDING:
        return 'secondary';
      default:
        return 'default';
    }
  };

  return (
    <Button
      onClick={handleFollowClick}
      variant={buttonVariant()}
      className="min-w-[100px]"
      disabled={isLoadingFollowStatus}
    >
      {buttonText()}
    </Button>
  );
};

---

§ 18. SRC/COMPONENTS/SOCIAL/USER-LIST.TSX (80 RIGHE)
typescript
import React from 'react';
import { Profile } from '@prisma/client';
import { ProfileCard } from './profile-card';
import { Loader2Icon } from 'lucide-react';
import { InfiniteQueryHookResult } from '~/types/trpc'; // Custom type for infinite query hook

interface UserListProps {
  queryHook: InfiniteQueryHookResult<Profile>;
  title: string;
  emptyMessage?: string;
}

export const UserList: React.FC<UserListProps> = ({ queryHook, title, emptyMessage = "No users found." }) => {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isLoading, isError, error } = queryHook;

  const allUsers = data?.pages.flatMap((page) => page.items) || [];

  if (isLoading) {
    return (
      <div className="flex justify-center items-center p-8">
        <Loader2Icon className="h-8 w-8 animate-spin text-blue-500" />
      </div>
    );
  }

  if (isError) {
    return <div className="p-4 text-red-500">Error loading {title}: {error?.message}</div>;
  }

  if (allUsers.length === 0) {
    return (
      <div className="p-8 text-center text-gray-500 dark:text-gray-400">
        {emptyMessage}
      </div>
    );
  }

  return (
    <div className="bg-white dark:bg-gray-800 shadow-sm rounded-lg">
      <h2 className="text-xl font-bold p-4 border-b border-gray-200 dark:border-gray-700">{title}</h2>
      <div>
        {allUsers.map((profile) => (
          <ProfileCard key={profile.id} profile={profile} />
        ))}
        {hasNextPage && (
          <div className="flex justify-center p-4">
            {isFetchingNextPage ? (
              <Loader2Icon className="h-6 w-6 animate-spin text-blue-500" />
            ) : (
              <button
                onClick={() => fetchNextPage()}
                disabled={!hasNextPage || isFetchingNextPage}
                className="text-blue-500 hover:underline"
              >
                Load more
              </button>
            )}
          </div>
        )}
        {!hasNextPage && allUsers.length > 0 && (
          <div className="p-4 text-center text-gray-500 dark:text-gray-400">
            End of list.
          </div>
        )}
      </div>
    </div>
  );
};

// Define InfiniteQueryHookResult type (adjust based on your actual tRPC setup)
type InfiniteQueryHookResult<T> = {
  data: { pages: { items: T[]; nextCursor?: string | null }[] } | undefined;
  fetchNextPage: () => Promise<any>;
  hasNextPage: boolean | undefined;
  isFetchingNextPage: boolean;
  isLoading: boolean;
  isError: boolean;
  error: any;
};

---

§ 19. SRC/APP/(SOCIAL)/FEED/PAGE.TSX (60 RIGHE)
typescript
'use client';

import React from 'react';
import { useSession } from 'next-auth/react';
import { PostComposer } from '~/components/social/post-composer';
import { Feed } from '~/components/social/feed';
import { useFeed } from '~/hooks/use-feed';
import { Loader2Icon } from 'lucide-react';

export default function HomePage() {
  const { data: session, status } = useSession();
  const { useHomeFeed } = useFeed();

  const homeFeedQuery = useHomeFeed();

  if (status === 'loading') {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <Loader2Icon className="h-10 w-10 animate-spin text-blue-500" />
      </div>
    );
  }

  if (!session) {
    return (
      <div className="p-8 text-center text-gray-600 dark:text-gray-300">
        Please sign in to view your personalized feed.
      </div>
    );
  }

  return (
    <div className="max-w-2xl mx-auto border-x border-gray-200 dark:border-gray-700">
      <h1 className="text-xl font-bold p-4 border-b border-gray-200 dark:border-gray-700">Home</h1>
      <PostComposer />
      <Feed
        queryHook={homeFeedQuery}
        viewerProfileId={session.user.profileId}
        emptyMessage="Your feed is empty. Start following people or post something!"
      />
    </div>
  );
}

---

§ 20. SRC/APP/(SOCIAL)/EXPLORE/PAGE.TSX (60 RIGHE)
typescript
'use client';

import React from 'react';
import { useSession } from 'next-auth/react';
import { Feed } from '~/components/social/feed';
import { useFeed } from '~/hooks/use-feed';
import { Loader2Icon } from 'lucide-react';

export default function ExplorePage() {
  const { data: session, status } = useSession();
  const { useExploreFeed } = useFeed();

  const exploreFeedQuery = useExploreFeed();

  if (status === 'loading') {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <Loader2Icon className="h-10 w-10 animate-spin text-blue-500" />
      </div>
    );
  }

  const viewerProfileId = session?.user?.profileId;

  return (
    <div className="max-w-2xl mx-auto border-x border-gray-200 dark:border-gray-700">
      <h1 className="text-xl font-bold p-4 border-b border-gray-200 dark:border-gray-700">Explore</h1>
      <Feed
        queryHook={exploreFeedQuery}
        viewerProfileId={viewerProfileId}
        emptyMessage="No popular posts to display right now. Check back later!"
      />
    </div>
  );
}

---

§ 21. SRC/APP/(SOCIAL)/[USERNAME]/PAGE.TSX (100 RIGHE)
typescript
'use client';

import React from 'react';
import { useParams } from 'next/navigation';
import { useSession } from 'next-auth/react';
import { ProfileHeader } from '~/components/social/profile-header';
import { Feed } from '~/components/social/feed';
import { useProfile } from '~/hooks/use-profile';
import { useFeed } from '~/hooks/use-feed';
import { Loader2Icon } from 'lucide-react';

export default function ProfilePage() {
  const params = useParams();
  const username = params.username as string;
  const { data: session, status } = useSession();

  const { profile, isLoadingProfile, profileError } = useProfile(username);
  const { useProfileFeed } = useFeed();

  const profileFeedQuery = useProfileFeed(profile?.id || '', session?.user?.profileId, { filterBy: 'all' });

  if (isLoadingProfile || status === 'loading') {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <Loader2Icon className="h-10 w-10 animate-spin text-blue-500" />
      </div>
    );
  }

  if (profileError || !profile) {
    return (
      <div className="max-w-2xl mx-auto p-8 text-center text-red-500">
        Error loading profile or profile not found.
      </div>
    );
  }

  const viewerProfileId = session?.user?.profileId;

  return (
    <div className="max-w-2xl mx-auto border-x border-gray-200 dark:border-gray-700">
      <ProfileHeader profile={profile} onEdit={() => console.log('Edit profile')} />
      <div className="mt-4">
        <h2 className="text-xl font-bold p-4 border-b border-gray-200 dark:border-gray-700">Posts</h2>
        <Feed
          queryHook={profileFeedQuery}
          viewerProfileId={viewerProfileId}
          emptyMessage={`${profile.displayName} hasn't posted anything yet.`}
        />
      </div>
    </div>
  );
}

---

§ 22. SRC/APP/(SOCIAL)/POST/[ID]/PAGE.TSX (80 RIGHE)
typescript
'use client';

import React from 'react';
import { useParams } from 'next/navigation';
import { useSession } from 'next-auth/react';
import { PostCard } from '~/components/social/post-card';
import { usePosts } from '~/hooks/use-posts';
import { Loader2Icon } from 'lucide-react';

export default function SinglePostPage() {
  const params = useParams();
  const postId = params.id as string;
  const { data: session, status } = useSession();

  const { getPostByIdQuery } = usePosts();
  const { data: post, isLoading, error } = getPostByIdQuery(postId);

  if (status === 'loading' || isLoading) {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <Loader2Icon className="h-10 w-10 animate-spin text-blue-500" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="max-w-2xl mx-auto p-8 text-center text-red-500">
        Error loading post: {error.message}
      </div>
    );
  }

  if (!post) {
    return (
      <div className="max-w-2xl mx-auto p-8 text-center text-gray-500">
        Post not found.
      </div>
    );
  }

  const viewerProfileId = session?.user?.profileId;

  return (
    <div className="max-w-2xl mx-auto border-x border-gray-200 dark:border-gray-700">
      <h1 className="text-xl font-bold p-4 border-b border-gray-200 dark:border-gray-700">Post</h1>
      <PostCard post={post} viewerProfileId={viewerProfileId} />
      {/* TODO: Add comments section here */}
      <div className="p-4 text-gray-500 dark:text-gray-400">Comments section coming soon...</div>
    </div>
  );
}

---

§ 23. TESTS/SOCIAL.TEST.TS (200 RIGHE)
typescript
import { PrismaClient, Profile, Post, Follow, FollowStatus } from '@prisma/client';
import { ProfileService } from '~/server/services/profile-service';
import { PostService } from '~/server/services/post-service';
import { FollowService } from '~/server/services/follow-service';
import { DeepMockProxy, mockDeep, mockReset } from 'jest-mock-extended';

// Mock Prisma Client
const prisma = mockDeep<PrismaClient>();
jest.mock('@prisma/client', () => ({
  __esModule: true,
  PrismaClient: jest.fn(() => prisma),
  PostVisibility: { PUBLIC: 'PUBLIC', FOLLOWERS_ONLY: 'FOLLOWERS_ONLY', PRIVATE: 'PRIVATE' },
  FollowStatus: { PENDING: 'PENDING', ACCEPTED: 'ACCEPTED', REJECTED: 'REJECTED' },
  NotificationType: { LIKE: 'LIKE', COMMENT: 'COMMENT', FOLLOW: 'FOLLOW', REPOST: 'REPOST', MENTION: 'MENTION' },
}));

// Mock NotificationService (used by PostService and FollowService)
jest.mock('~/server/services/notification-service', () => ({
  NotificationService: jest.fn().mockImplementation(() => ({
    create: jest.fn(),
  })),
}));

describe('Social Services Integration Tests', () => {
  let profileService: ProfileService;
  let postService: PostService;
  let followService: FollowService;

  let user1Profile: Profile;
  let user2Profile: Profile;
  let user3ProfilePrivate: Profile;

  beforeEach(async () => {
    mockReset(prisma); // Reset mocks before each test
    profileService = new ProfileService();
    postService = new PostService();
    followService = new FollowService();

    // Setup mock data for profiles
    user1Profile = {
      id: 'profile1', userId: 'user1', username: 'userone', displayName: 'User One', bio: 'Bio 1',
      avatar: null, coverImage: null, website: null, location: null,
      followersCount: 0, followingCount: 0, postsCount: 0,
      isPrivate: false, isVerified: false, createdAt: new Date(), updatedAt: new Date(),
    };
    user2Profile = {
      id: 'profile2', userId: 'user2', username: 'usertwo', displayName: 'User Two', bio: 'Bio 2',
      avatar: null, coverImage: null, website: null, location: null,
      followersCount: 0, followingCount: 0, postsCount: 0,
      isPrivate: false, isVerified: false, createdAt: new Date(), updatedAt: new Date(),
    };
    user3ProfilePrivate = {
      id: 'profile3', userId: 'user3', username: 'userthree', displayName: 'User Three', bio: 'Bio 3',
      avatar: null, coverImage: null, website: null, location: null,
      followersCount: 0, followingCount: 0, postsCount: 0,
      isPrivate: true, isVerified: false, createdAt: new Date(), updatedAt: new Date(),
    };

    // Mock profile service dependencies for other services
    jest.spyOn(profileService, 'getById').mockImplementation(async (id: string) => {
      if (id === user1Profile.id) return user1Profile;
      if (id === user2Profile.id) return user2Profile;
      if (id === user3ProfilePrivate.id) return user3ProfilePrivate;
      return null;
    });
    jest.spyOn(profileService, 'getByUsername').mockImplementation(async (username: string) => {
      if (username === user1Profile.username) return user1Profile;
      if (username === user2Profile.username) return user2Profile;
      if (username === user3ProfilePrivate.username) return user3ProfilePrivate;
      return null;
    });
    jest.spyOn(profileService, 'incrementStat').mockResolvedValue(undefined);
  });

  describe('ProfileService', () => {
    it('should create a profile', async () => {
      prisma.profile.create.mockResolvedValue(user1Profile);
      const newProfile = await profileService.create('user1', {
        username: 'userone', displayName: 'User One', bio: 'Bio 1'
      });
      expect(newProfile).toEqual(user1Profile);
      expect(prisma.profile.create).toHaveBeenCalledWith(expect.any(Object));
    });

    it('should check username availability', async () => {
      prisma.profile.findUnique.mockResolvedValue(null);
      const available = await profileService.isUsernameAvailable('newuser');
      expect(available).toBe(true);

      prisma.profile.findUnique.mockResolvedValue(user1Profile);
      const notAvailable = await profileService.isUsernameAvailable('userone');
      expect(notAvailable).toBe(false);
    });
  });

  describe('FollowService', () => {
    it('should allow a user to follow another public user', async () => {
      prisma.follow.create.mockResolvedValue({
        id: 'f1', followerId: user1Profile.id, followingId: user2Profile.id,
        status: FollowStatus.ACCEPTED, createdAt: new Date()
      });
      const follow = await followService.follow(user1Profile.id, user2Profile.id);
      expect(follow.status).toBe(FollowStatus.ACCEPTED);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user1Profile.id, 'following', 1);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user2Profile.id, 'followers', 1);
    });

    it('should create a pending follow request for a private user', async () => {
      prisma.follow.create.mockResolvedValue({
        id: 'f2', followerId: user1Profile.id, followingId: user3ProfilePrivate.id,
        status: FollowStatus.PENDING, createdAt: new Date()
      });
      const follow = await followService.follow(user1Profile.id, user3ProfilePrivate.id);
      expect(follow.status).toBe(FollowStatus.PENDING);
      expect(profileService.incrementStat).not.toHaveBeenCalledWith(user1Profile.id, 'following', 1);
      expect(profileService.incrementStat).not.toHaveBeenCalledWith(user3ProfilePrivate.id, 'followers', 1);
    });

    it('should accept a pending follow request', async () => {
      const pendingFollow = {
        id: 'f2', followerId: user1Profile.id, followingId: user3ProfilePrivate.id,
        status: FollowStatus.PENDING, createdAt: new Date()
      };
      prisma.follow.update.mockResolvedValue({ ...pendingFollow, status: FollowStatus.ACCEPTED });

      const acceptedFollow = await followService.acceptFollowRequest(pendingFollow.id);
      expect(acceptedFollow.status).toBe(FollowStatus.ACCEPTED);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user1Profile.id, 'following', 1);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user3ProfilePrivate.id, 'followers', 1);
    });

    it('should unfollow a user', async () => {
      prisma.follow.delete.mockResolvedValue({
        id: 'f1', followerId: user1Profile.id, followingId: user2Profile.id,
        status: FollowStatus.ACCEPTED, createdAt: new Date()
      });
      await followService.unfollow(user1Profile.id, user2Profile.id);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user1Profile.id, 'following', -1);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user2Profile.id, 'followers', -1);
    });
  });

  describe('PostService', () => {
    it('should create a post and increment author post count', async () => {
      const newPost: Post = {
        id: 'post1', authorId: user1Profile.id, content: 'Hello world!', images: [], videos: [],
        likesCount: 0, commentsCount: 0, sharesCount: 0, viewsCount: 0,
        visibility: 'PUBLIC', isPinned: false, repostOfId: null,
        createdAt: new Date(), updatedAt: new Date(), deletedAt: null,
      };
      prisma.post.create.mockResolvedValue(newPost);

      const createdPost = await postService.create(user1Profile.id, { content: 'Hello world!' });
      expect(createdPost).toEqual(newPost);
      expect(profileService.incrementStat).toHaveBeenCalledWith(user1Profile.id, 'posts', 1);
    });

    it('should like a post and increment like count', async () => {
      const post: Post = {
        id: 'post1', authorId: user2Profile.id, content: 'Test post', images: [], videos: [],
        likesCount: 0, commentsCount: 0, sharesCount: 0, viewsCount: 0,
        visibility: 'PUBLIC', isPinned: false, repostOfId: null,
        createdAt: new Date(), updatedAt: new Date(), deletedAt: null,
      };
      prisma.like.create.mockResolvedValue({
        id: 'l1', profileId: user1Profile.id, postId: post.id, createdAt: new Date()
      });
      prisma.post.update.mockResolvedValue({ ...post, likesCount: 1 });
      prisma.post.findUnique.mockResolvedValue(post); // For getById call in like method

      await postService.like(post.id, user1Profile.id);
      expect(prisma.like.create).toHaveBeenCalledWith(expect.any(Object));
      expect(prisma.post.update).toHaveBeenCalledWith(expect.objectContaining({
        where: { id: post.id },
        data: { likesCount: { increment: 1 } }
      }));
    });

    it('should delete a post and decrement author post count', async () => {
      const post: Post = {
        id: 'post1', authorId: user1Profile.id, content: 'Hello world!', images: [], videos: [],
        likesCount: 0, commentsCount: 0, sharesCount: 0, viewsCount: 0,
        visibility: 'PUBLIC', isPinned: false, repostOfId: null,
        createdAt: new Date(), updatedAt: new Date(), deletedAt: null,
      };
      prisma.post.update.mockResolvedValue({ ...post, deletedAt: new Date() });

      await postService.delete(post.id);
      expect(prisma.post.update).toHaveBeenCalledWith(expect.objectContaining({
        where: { id: post.id },
        data: { deletedAt: expect.any(Date) }
      }));
      expect(profileService.incrementStat).toHaveBeenCalledWith(user1Profile.id, 'posts', -1);
    });
  });
});

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 33582_