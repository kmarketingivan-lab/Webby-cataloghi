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
_Modello: gemini-2.5-flash_