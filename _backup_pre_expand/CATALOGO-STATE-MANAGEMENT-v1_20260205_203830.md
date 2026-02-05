# CATALOGO STATE MANAGEMENT v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: State Management React, Zustand, Jotai, TanStack Query, Redux Toolkit

---

## 1. SOLUTION COMPARISON

| Library             | Bundle Size | DevTools         | SSR | Atomic | Best For            | Learning |
| ------------------- | ----------- | ---------------- | --- | ------ | ------------------- | -------- |
| useState/useReducer | 0 KB        | React DevTools   | ✅  | N/A    | Component state     | ⭐       |
| React Context       | 0 KB        | React DevTools   | ✅  | ❌     | Theme, auth         | ⭐       |
| Zustand             | 1.5 KB      | ✅ Redux DevTools | ✅  | ❌     | App state simple    | ⭐⭐     |
| Jotai               | 3 KB        | ✅ Jotai DevTools | ✅  | ✅     | Atomic, derived     | ⭐⭐     |
| Redux Toolkit       | 10 KB       | ✅ Redux DevTools | ✅  | ❌     | Complex, enterprise | ⭐⭐⭐   |
| Valtio              | 4 KB        | ⚠️              | ✅  | ❌     | Proxy-based         | ⭐⭐     |
| TanStack Query      | 13 KB       | ✅ Query DevTools | ✅  | ❌     | Server state        | ⭐⭐     |
| Recoil              | 20 KB       | ✅               | ⚠️ | ✅     | Meta (legacy)       | ⭐⭐⭐   |

### 1.1 Feature Deep Comparison

| Feature       | Zustand      | Jotai             | Redux TK        | TanStack Query |
| ------------- | ------------ | ----------------- | --------------- | -------------- |
| Boilerplate   | Minimal      | Minimal           | Medium          | Medium         |
| TypeScript    | ⭐⭐⭐⭐⭐   | ⭐⭐⭐⭐⭐        | ⭐⭐⭐⭐        | ⭐⭐⭐⭐⭐     |
| Devtools      | Redux DT     | Jotai DT          | Redux DT        | Query DT       |
| Middleware    | ✅           | ✅                | ✅              | ❌             |
| Persist       | ✅ Built-in  | ✅ atomWithStorage | ✅ redux-persist | ⚠️            |
| Immer         | ✅ Middleware | ⚠️ Manual        | ✅ Built-in     | N/A            |
| Async         | ✅ Native    | ✅ Suspense       | ✅ Thunks       | ✅ Core feature |
| React Native  | ✅           | ✅                | ✅              | ✅             |
| Bundle impact | +1.5KB       | +3KB              | +10KB           | +13KB          |

---

## 2. DECISION FLOWCHART

```
START: Che tipo di stato stai gestendo?
│
├─► "Stato del SERVER (API data, cache)"
│   └─► TanStack Query (o SWR)
│       - Caching automatico
│       - Background refetch
│       - Optimistic updates
│
├─► "Stato LOCALE del componente"
│   ├─► Semplice (toggle, input)?
│   │   └─► useState
│   └─► Complesso (reducer logic)?
│       └─► useReducer
│
├─► "Stato CONDIVISO tra componenti"
│   ├─► Cambia raramente (theme, locale)?
│   │   └─► React Context
│   ├─► Molti subscribers con update frequenti?
│   │   ├─► Preferisci store globale?
│   │   │   └─► Zustand
│   │   └─► Preferisci stato atomico?
│   │       └─► Jotai
│   └─► Team grande, enterprise?
│       └─► Redux Toolkit
│
└─► "URL State (filters, pagination)"
    └─► nuqs o native URLSearchParams
```

---

## 3. ZUSTAND PATTERNS

### 3.1 Basic Store

```typescript
// src/stores/counter.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  incrementBy: (amount: number) => void;
}

export const useCounterStore = create<CounterState>()(
  devtools(
    persist(
      (set) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 })),
        decrement: () => set((state) => ({ count: state.count - 1 })),
        reset: () => set({ count: 0 }),
        incrementBy: (amount) => set((state) => ({ count: state.count + amount })),
      }),
      {
        name: 'counter-storage',
        partialize: (state) => ({ count: state.count }),
      }
    ),
    { name: 'CounterStore' }
  )
);

// Usage in component
// const count = useCounterStore((state) => state.count);
// const increment = useCounterStore((state) => state.increment);
```

### 3.2 Slices Pattern (Large Apps)

```typescript
// src/stores/slices/userSlice.ts
import { StateCreator } from 'zustand';

export interface UserSlice {
  user: User | null;
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  logout: () => void;
}

export const createUserSlice: StateCreator<
  UserSlice & SettingsSlice,
  [],
  [],
  UserSlice
> = (set) => ({
  user: null,
  isAuthenticated: false,
  setUser: (user) => set({ user, isAuthenticated: !!user }),
  logout: () => set({ user: null, isAuthenticated: false }),
});

// src/stores/slices/settingsSlice.ts
export interface SettingsSlice {
  theme: 'light' | 'dark' | 'system';
  language: string;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  setLanguage: (language: string) => void;
}

export const createSettingsSlice: StateCreator<
  UserSlice & SettingsSlice,
  [],
  [],
  SettingsSlice
> = (set) => ({
  theme: 'system',
  language: 'en',
  setTheme: (theme) => set({ theme }),
  setLanguage: (language) => set({ language }),
});

// src/stores/index.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

type AppStore = UserSlice & SettingsSlice;

export const useAppStore = create<AppStore>()(
  devtools(
    persist(
      (...a) => ({
        ...createUserSlice(...a),
        ...createSettingsSlice(...a),
      }),
      {
        name: 'app-storage',
        partialize: (state) => ({
          theme: state.theme,
          language: state.language,
        }),
      }
    ),
    { name: 'AppStore' }
  )
);
```

### 3.3 Async Actions

```typescript
import { create } from 'zustand';

interface UserState {
  users: User[];
  isLoading: boolean;
  error: string | null;
  fetchUsers: () => Promise<void>;
}

export const useUserStore = create<UserState>((set) => ({
  users: [],
  isLoading: false,
  error: null,
  fetchUsers: async () => {
    set({ isLoading: true, error: null });
    try {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Failed to fetch');
      const users = await response.json();
      set({ users, isLoading: false });
    } catch (error) {
      set({ 
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false 
      });
    }
  },
}));
```

### 3.4 Immer Middleware

```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface TodoState {
  todos: Todo[];
  addTodo: (text: string) => void;
  toggleTodo: (id: string) => void;
  removeTodo: (id: string) => void;
}

export const useTodoStore = create<TodoState>()(
  immer((set) => ({
    todos: [],
    addTodo: (text) =>
      set((state) => {
        state.todos.push({ id: crypto.randomUUID(), text, completed: false });
      }),
    toggleTodo: (id) =>
      set((state) => {
        const todo = state.todos.find((t) => t.id === id);
        if (todo) todo.completed = !todo.completed;
      }),
    removeTodo: (id) =>
      set((state) => {
        state.todos = state.todos.filter((t) => t.id !== id);
      }),
  }))
);
```

### 3.5 Selectors Optimization

```typescript
import { useAppStore } from './index';
import { useMemo } from 'react';

// Selective subscription to prevent unnecessary re-renders
export function useActiveUsers() {
  const users = useAppStore((state) => state.users);
  return useMemo(() => users.filter((u) => u.isActive), [users]);
}
```

### 3.6 Persist Middleware

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface AuthState {
  token: string | null;
  setToken: (token: string | null) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      setToken: (token) => set({ token }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ token: state.token }),
      version: 1,
    }
  )
);
```

---

## 4. JOTAI PATTERNS

### 4.1 Primitive Atoms

```typescript
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

export const countAtom = atom(0);

export function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

### 4.2 Derived Atoms

```typescript
import { atom } from 'jotai';

export const firstNameAtom = atom('John');
export const lastNameAtom = atom('Doe');

export const fullNameAtom = atom((get) => {
  const first = get(firstNameAtom);
  const last = get(lastNameAtom);
  return `${first} ${last}`;
});
```

### 4.3 Async Atoms

```typescript
import { atom } from 'jotai';

export const userAtom = atom(async () => {
  const res = await fetch('/api/user');
  return res.json() as Promise<User>;
});

export function UserProfile() {
  const user = useAtomValue(userAtom); // Suspense
  return <div>{user.name}</div>;
}
```

### 4.4 atomWithStorage

```typescript
import { atomWithStorage, createJSONStorage } from 'jotai/utils';

export const themeAtom = atomWithStorage<'light' | 'dark'>('theme', 'light');
export const sessionAtom = atomWithStorage(
  'session-data', 
  null, 
  createJSONStorage(() => sessionStorage)
);
```

### 4.5 atomWithQuery

```typescript
import { atomWithQuery } from 'jotai-tanstack-query';

export const usersQueryAtom = atomWithQuery(() => ({
  queryKey: ['users'],
  queryFn: async () => {
    const res = await fetch('/api/users');
    return res.json() as Promise<User[]>;
  },
}));
```

---

## 5. TANSTACK QUERY PATTERNS

### 5.1 Basic Query

```typescript
import { useQuery } from '@tanstack/react-query';

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await fetch('/api/users');
      if (!res.ok) throw new Error('Failed');
      return res.json() as Promise<User[]>;
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <ul>{data?.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### 5.2 Mutations & Optimistic Updates

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateUser() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: async (user: { name: string }) => {
      const res = await fetch('/api/users', { 
        method: 'POST', 
        body: JSON.stringify(user) 
      });
      return res.json();
    },
    onMutate: async (newUser) => {
      await queryClient.cancelQueries(['users']);
      const prev = queryClient.getQueryData<User[]>(['users']);
      queryClient.setQueryData(['users'], (old = []) => [...old, newUser as any]);
      return { prev };
    },
    onError: (_err, _newUser, context) => {
      if (context?.prev) queryClient.setQueryData(['users'], context.prev);
    },
    onSettled: () => queryClient.invalidateQueries(['users']),
  });

  return <button onClick={() => mutation.mutate({ name: 'New User' })}>Add</button>;
}
```

### 5.3 Infinite Queries

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';

function PostsList() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: async ({ pageParam = '' }) => {
      const res = await fetch(`/api/posts?cursor=${pageParam}`);
      return res.json() as Promise<{ posts: Post[]; nextCursor: string | null }>;
    },
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });

  return (
    <>
      {data?.pages.map((p, i) => (
        <React.Fragment key={i}>
          {p.posts.map((post) => <div key={post.id}>{post.title}</div>)}
        </React.Fragment>
      ))}
      {hasNextPage && (
        <button onClick={() => fetchNextPage()} disabled={isFetchingNextPage}>
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </>
  );
}
```

### 5.4 Cache Invalidation

```typescript
import { useQueryClient } from '@tanstack/react-query';

function RefreshUsersButton() {
  const queryClient = useQueryClient();

  const handleRefresh = () => {
    // Invalidate to trigger refetch
    queryClient.invalidateQueries({ queryKey: ['users'] });
    
    // Reset to initial state
    queryClient.resetQueries({ queryKey: ['users'] });
    
    // Remove from cache
    queryClient.removeQueries({ queryKey: ['users'] });
  };

  return <button onClick={handleRefresh}>Refresh</button>;
}
```

### 5.5 Dependent Queries

```typescript
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }: { userId: string | null }) {
  const userQuery = useQuery({
    queryKey: ['user', userId],
    queryFn: async () => {
      const res = await fetch(`/api/users/${userId}`);
      return res.json() as Promise<User>;
    },
    enabled: !!userId, // Only run if userId is truthy
  });

  if (!userId) return <div>No user selected</div>;
  if (userQuery.isLoading) return <div>Loading...</div>;
  return <div>User: {userQuery.data?.name}</div>;
}
```

### 5.6 Prefetching

```typescript
import { useQueryClient } from '@tanstack/react-query';

function PrefetchUser({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const prefetch = async () => {
    await queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: async () => {
        const res = await fetch(`/api/users/${userId}`);
        return res.json();
      },
      staleTime: 5 * 60 * 1000, // 5 minutes
    });
  };

  return <button onMouseEnter={prefetch}>Hover to prefetch</button>;
}
```

### 5.7 SSR + React Query (Next.js)

```typescript
// pages/users.tsx
import { dehydrate, QueryClient } from '@tanstack/react-query';
import { GetServerSideProps } from 'next';
import { useQuery } from '@tanstack/react-query';

async function fetchUsers() {
  const res = await fetch('https://api.example.com/users');
  return res.json() as Promise<User[]>;
}

export const getServerSideProps: GetServerSideProps = async () => {
  const queryClient = new QueryClient();
  await queryClient.prefetchQuery(['users'], fetchUsers);

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
};

export default function UsersPage() {
  const { data, isLoading } = useQuery(['users'], fetchUsers);
  if (isLoading) return <div>Loading...</div>;
  return <ul>{data?.map((user) => <li key={user.id}>{user.name}</li>)}</ul>;
}
```

---

## 6. INTEGRATION PATTERNS

### 6.1 Zustand + React Query

```typescript
import { create } from 'zustand';
import { useQuery } from '@tanstack/react-query';

interface UserStore {
  users: User[];
  setUsers: (users: User[]) => void;
}

export const useUserStore = create<UserStore>((set) => ({
  users: [],
  setUsers: (users) => set({ users }),
}));

export function UsersFetcher() {
  const setUsers = useUserStore((state) => state.setUsers);

  useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await fetch('/api/users');
      const data = await res.json() as User[];
      setUsers(data); // Sync with Zustand
      return data;
    },
    staleTime: 60000,
  });

  return null;
}
```

### 6.2 Jotai + React Query

```typescript
import { atomWithQuery } from 'jotai-tanstack-query';
import { useAtomValue } from 'jotai';

const usersQueryAtom = atomWithQuery(() => ({
  queryKey: ['users'],
  queryFn: async () => {
    const res = await fetch('/api/users');
    return res.json() as Promise<User[]>;
  },
}));

export function UsersList() {
  const users = useAtomValue(usersQueryAtom);
  return <ul>{users?.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### 6.3 DevTools + Logging

```typescript
import { create } from 'zustand';
import { devtools, subscribeWithSelector } from 'zustand/middleware';

export const useCounterStore = create<CounterState>()(
  devtools(
    subscribeWithSelector((set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 })),
    })),
    { name: 'CounterStore', enabled: process.env.NODE_ENV === 'development' }
  )
);

// Logging every state change
useCounterStore.subscribe(
  (state) => state.count,
  (count) => console.log('[Zustand]', count)
);
```

---

## 7. STATE MANAGEMENT CHECKLIST

```
□ Identificato tipo di stato (server vs client)
□ Scelta libreria appropriata per use case
□ TypeScript strict mode abilitato
□ DevTools configurati per development
□ Persist middleware per dati critici
□ Selectors ottimizzati per re-render
□ Error handling implementato
□ Loading states gestiti
□ Cache invalidation strategy definita
□ SSR compatibility verificata
□ Bundle size verificato
```

---

## 8. QUICK REFERENCE

### When to Use What

| Scenario                    | Solution          |
| --------------------------- | ----------------- |
| Simple component state      | useState          |
| Complex component logic     | useReducer        |
| Theme/locale (rare updates) | React Context     |
| Global app state            | Zustand           |
| Atomic/derived state        | Jotai             |
| Server data caching         | TanStack Query    |
| Enterprise/strict patterns  | Redux Toolkit     |
| URL state                   | nuqs/URLSearchParams |
