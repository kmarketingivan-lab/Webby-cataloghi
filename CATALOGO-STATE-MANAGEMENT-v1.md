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

9. ZUSTAND ADVANCED PATTERNS
Slice Pattern per Moduli Separati

Il pattern a slice permette di comporre store modulari e mantenibili, separando la logica in file distinti.

typescript
Copia
Scarica
// types/todo.ts
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

export interface TodoSlice {
  todos: Todo[];
  addTodo: (text: string) => void;
  toggleTodo: (id: string) => void;
  removeTodo: (id: string) => void;
}

export interface FilterSlice {
  filter: 'all' | 'active' | 'completed';
  setFilter: (filter: 'all' | 'active' | 'completed') => void;
}

// store/slices/useTodoSlice.ts
import { StateCreator } from 'zustand';
import { TodoSlice } from '@/types/todo';

const createTodoSlice: StateCreator<
  TodoSlice,
  [],
  [],
  TodoSlice
> = (set) => ({
  todos: [],
  addTodo: (text) =>
    set((state) => ({
      todos: [
        ...state.todos,
        { id: Date.now().toString(), text, completed: false },
      ],
    })),
  toggleTodo: (id) =>
    set((state) => ({
      todos: state.todos.map((todo) =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      ),
    })),
  removeTodo: (id) =>
    set((state) => ({
      todos: state.todos.filter((todo) => todo.id !== id),
    })),
});

// store/slices/useFilterSlice.ts
import { StateCreator } from 'zustand';
import { FilterSlice } from '@/types/todo';

const createFilterSlice: StateCreator<
  FilterSlice,
  [],
  [],
  FilterSlice
> = (set) => ({
  filter: 'all',
  setFilter: (filter) => set({ filter }),
});

// store/useStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { TodoSlice, FilterSlice } from '@/types/todo';
import { createTodoSlice } from './slices/useTodoSlice';
import { createFilterSlice } from './slices/useFilterSlice';

type Store = TodoSlice & FilterSlice;

export const useStore = create<Store>()(
  devtools(
    persist(
      (...a) => ({
        ...createTodoSlice(...a),
        ...createFilterSlice(...a),
      }),
      { name: 'todo-storage' }
    )
  )
);

// components/TodoList.tsx
'use client';
import { useStore } from '@/store/useStore';

export function TodoList() {
  const { todos, filter, toggleTodo } = useStore();
  
  const filteredTodos = todos.filter((todo) => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <ul>
      {filteredTodos.map((todo) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
Immer Middleware per State Immutabile
typescript
Copia
Scarica
// store/useImmerStore.ts
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface ProductState {
  products: Record<string, { quantity: number; price: number }>;
  addProduct: (id: string, price: number) => void;
  incrementQuantity: (id: string) => void;
  bulkUpdate: (updates: Array<{ id: string; price: number }>) => void;
}

export const useProductStore = create<ProductState>()(
  immer((set) => ({
    products: {},
    addProduct: (id, price) =>
      set((state) => {
        // Diretta mutazione con immer - più leggibile
        if (!state.products[id]) {
          state.products[id] = { quantity: 0, price };
        }
        state.products[id].quantity += 1;
      }),
    incrementQuantity: (id) =>
      set((state) => {
        if (state.products[id]) {
          state.products[id].quantity += 1;
        }
      }),
    bulkUpdate: (updates) =>
      set((state) => {
        updates.forEach(({ id, price }) => {
          if (state.products[id]) {
            state.products[id].price = price;
          }
        });
      }),
  }))
);

// components/ProductManager.tsx
'use client';
import { useProductStore } from '@/store/useImmerStore';

export function ProductManager() {
  const products = useProductStore((state) => state.products);
  const incrementQuantity = useProductStore((state) => state.incrementQuantity);

  return (
    <div>
      {Object.entries(products).map(([id, product]) => (
        <div key={id}>
          Product {id}: {product.quantity} × ${product.price}
          <button onClick={() => incrementQuantity(id)}>+</button>
        </div>
      ))}
    </div>
  );
}
Computed/Derived State con Selector
typescript
Copia
Scarica
// store/useCartStore.ts
import { create } from 'zustand';
import { shallow } from 'zustand/shallow';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  clearCart: () => void;
}

export const useCartStore = create<CartState>((set) => ({
  items: [],
  addItem: (item) =>
    set((state) => {
      const existing = state.items.find((i) => i.id === item.id);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.id === item.id
              ? { ...i, quantity: i.quantity + item.quantity }
              : i
          ),
        };
      }
      return { items: [...state.items, item] };
    }),
  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter((item) => item.id !== id),
    })),
  clearCart: () => set({ items: [] }),
}));

// Hook per derived state - memoizzazione automatica
export const useCartSummary = () => {
  return useCartStore(
    (state) => {
      const totalItems = state.items.reduce(
        (sum, item) => sum + item.quantity,
        0
      );
      const totalPrice = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
      const hasItems = state.items.length > 0;

      return {
        totalItems,
        totalPrice,
        hasItems,
        itemCount: state.items.length,
      };
    },
    shallow // shallow compare per evitare re-render non necessari
  );
};

// components/CartSummary.tsx
'use client';
import { useCartSummary } from '@/store/useCartStore';

export function CartSummary() {
  const { totalItems, totalPrice, hasItems } = useCartSummary();

  if (!hasItems) return <p>Cart is empty</p>;

  return (
    <div>
      <p>Items: {totalItems}</p>
      <p>Total: ${totalPrice.toFixed(2)}</p>
    </div>
  );
}
Zustand + React Query Integration
typescript
Copia
Scarica
// store/useUserStore.ts
import { create } from 'zustand';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { User, ApiResponse } from '@/types/user';

interface UserState {
  users: User[];
  selectedUserId: string | null;
  setSelectedUser: (id: string | null) => void;
}

const useUserStore = create<UserState>((set) => ({
  users: [],
  selectedUserId: null,
  setSelectedUser: (id) => set({ selectedUserId: id }),
}));

// Hook combinato che integra Zustand e React Query
export function useUserManagement() {
  const queryClient = useQueryClient();
  const { users, selectedUserId, setSelectedUser } = useUserStore();
  
  // React Query per fetch remoto
  const { data: remoteUsers, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: async (): Promise<User[]> => {
      const res = await fetch('/api/users');
      const data: ApiResponse<User[]> = await res.json();
      return data.data;
    },
    onSuccess: (data) => {
      // Sincronizza i dati remoti nello store Zustand
      useUserStore.setState({ users: data });
    },
  });

  // Mutation con optimistic update
  const addUserMutation = useMutation({
    mutationFn: async (user: Omit<User, 'id'>) => {
      const res = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(user),
      });
      return res.json();
    },
    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['users'] });
      
      // Snapshot del valore precedente
      const previousUsers = queryClient.getQueryData<User[]>(['users']);
      
      // Optimistic update nello Zustand store
      const optimisticUser: User = {
        ...newUser,
        id: `temp-${Date.now()}`,
      };
      
      useUserStore.setState((state) => ({
        users: [...state.users, optimisticUser],
      }));
      
      // Optimistic update in React Query cache
      queryClient.setQueryData<User[]>(['users'], (old = []) => [
        ...old,
        optimisticUser,
      ]);
      
      return { previousUsers };
    },
    onError: (err, newUser, context) => {
      // Rollback in caso di errore
      if (context?.previousUsers) {
        queryClient.setQueryData(['users'], context.previousUsers);
        useUserStore.setState({ users: context.previousUsers });
      }
    },
    onSettled: () => {
      // Refetch per sincronizzazione
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  return {
    users: remoteUsers || users,
    isLoading,
    selectedUserId,
    setSelectedUser,
    addUser: addUserMutation.mutate,
    isAdding: addUserMutation.isPending,
  };
}
Persist con Encryption e Migration
typescript
Copia
Scarica
// store/useSecureStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage, StateStorage } from 'zustand/middleware';
import CryptoJS from 'crypto-js';

// Encryption utilities
const SECRET_KEY = process.env.NEXT_PUBLIC_STORAGE_KEY || 'fallback-secret';

const encryptData = (data: string): string => {
  return CryptoJS.AES.encrypt(data, SECRET_KEY).toString();
};

const decryptData = (ciphertext: string): string => {
  const bytes = CryptoJS.AES.decrypt(ciphertext, SECRET_KEY);
  return bytes.toString(CryptoJS.enc.Utf8);
};

// Custom storage con encryption
const encryptedStorage: StateStorage = {
  getItem: (name: string): string | null => {
    if (typeof window === 'undefined') return null;
    
    try {
      const encrypted = localStorage.getItem(name);
      if (!encrypted) return null;
      
      const decrypted = decryptData(encrypted);
      return decrypted;
    } catch (error) {
      console.error('Decryption error:', error);
      return null;
    }
  },
  setItem: (name: string, value: string): void => {
    if (typeof window === 'undefined') return;
    
    try {
      const encrypted = encryptData(value);
      localStorage.setItem(name, encrypted);
    } catch (error) {
      console.error('Encryption error:', error);
    }
  },
  removeItem: (name: string): void => {
    if (typeof window === 'undefined') return;
    localStorage.removeItem(name);
  },
};

// Migration schema
type Migrations = Record<number, (state: any) => any>;

interface AppSettings {
  version: number;
  theme: 'light' | 'dark' | 'system';
  language: string;
  notifications: boolean;
  fontSize: number;
}

interface SettingsState extends AppSettings {
  updateSetting: <K extends keyof AppSettings>(
    key: K,
    value: AppSettings[K]
  ) => void;
  resetSettings: () => void;
}

const migrations: Migrations = {
  1: (state) => ({ ...state, version: 1 }), // Initial version
  2: (state) => ({
    ...state,
    version: 2,
    fontSize: state.fontSize || 16, // Add default fontSize
  }),
  3: (state) => ({
    ...state,
    version: 3,
    theme: state.theme || 'system', // Ensure theme exists
  }),
};

const initialSettings: AppSettings = {
  version: 3,
  theme: 'system',
  language: 'en',
  notifications: true,
  fontSize: 16,
};

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      ...initialSettings,
      updateSetting: (key, value) =>
        set((state) => ({ ...state, [key]: value })),
      resetSettings: () => set({ ...initialSettings }),
    }),
    {
      name: 'app-settings',
      storage: createJSONStorage(() => encryptedStorage),
      version: 3,
      migrate: (persistedState: any, version: number) => {
        let state = persistedState as AppSettings;
        
        // Apply migrations in order
        for (let i = version; i < 3; i++) {
          const migration = migrations[i + 1];
          if (migration) {
            state = migration(state);
          }
        }
        
        return state as SettingsState;
      },
      // Partialize per salvare solo campi specifici
      partialize: (state) => ({
        theme: state.theme,
        language: state.language,
        notifications: state.notifications,
        fontSize: state.fontSize,
        version: state.version,
      }),
    }
  )
);

// Hook di utilità
export function useSettings() {
  const settings = useSettingsStore();
  
  // Computed values
  const isDarkMode = settings.theme === 'dark' || 
    (settings.theme === 'system' && 
     typeof window !== 'undefined' && 
     window.matchMedia('(prefers-color-scheme: dark)').matches);
  
  return {
    ...settings,
    isDarkMode,
  };
}
Tabella: 6 Middleware Zustand
Nome Middleware	Descrizione	Use Case	Bundle Impact
persist	Salva lo store in localStorage/sessionStorage	Impostazioni utente, carrello, preferenze	✅ ~2KB (gzipped)
immer	Permette mutazioni immutabili con sintassi mutabile	Stato complesso con nested objects	✅ ~4KB (gzipped)
devtools	Integrazione con Redux DevTools Browser Extension	Debug durante sviluppo	✅ ~1.5KB (development only)
redux	Adapter per reducer Redux-style	Migrazione da Redux esistente	✅ ~0.5KB (gzipped)
subscribeWithSelector	Subscribe a porzioni specifiche dello stato	Logging, analytics, sync esterno	✅ ~0.3KB (gzipped)
combine	Combina multipli store in uno	Architetture a moduli separati	✅ ~0.2KB (gzipped)
10. JOTAI ADVANCED PATTERNS
atomFamily per Stato Parametrico
typescript
Copia
Scarica
// atoms/todoAtoms.ts
import { atomFamily } from 'jotai/utils';
import { atom } from 'jotai';
import { fetchTodoById, updateTodo } from '@/lib/api';

// atomFamily per todo items con parametro id
export const todoAtomFamily = atomFamily((id: string) =>
  atom(
    async (get) => {
      // Async read - fetch iniziale
      const todo = await fetchTodoById(id);
      return todo;
    },
    async (get, set, update: { text?: string; completed?: boolean }) => {
      // Async write - optimistic update pattern
      const current = get(todoAtomFamily(id));
      const optimisticTodo = { ...current, ...update };
      
      // Set ottimistico locale
      set(todoAtomFamily(id), optimisticTodo);
      
      try {
        // Sync con backend
        await updateTodo(id, update);
      } catch (error) {
        // Rollback in caso di errore
        set(todoAtomFamily(id), current);
        throw error;
      }
    }
  )
);

// atoms/todoListAtoms.ts
import { atom } from 'jotai';
import { atomFamily } from 'jotai/utils';

// Filtri parametrici
export const filterAtom = atom<'all' | 'active' | 'completed'>('all');

// Lista di ID dei todo (lightweight reference)
export const todoIdsAtom = atom<string[]>([]);

// Atom derivato per lista filtrata
export const filteredTodosAtom = atom(async (get) => {
  const ids = get(todoIdsAtom);
  const filter = get(filterAtom);
  
  const todos = await Promise.all(
    ids.map((id) => get(todoAtomFamily(id)))
  );
  
  return todos.filter((todo) => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
});

// Operazioni batch
export const markAllAsCompletedAtom = atom(
  null,
  async (get, set) => {
    const ids = get(todoIdsAtom);
    
    await Promise.all(
      ids.map(async (id) => {
        const todo = get(todoAtomFamily(id));
        if (!todo.completed) {
          await set(todoAtomFamily(id), { completed: true });
        }
      })
    );
  }
);
atomWithQuery (TanStack Query Integration)
typescript
Copia
Scarica
// atoms/queryAtoms.ts
import { atom } from 'jotai';
import { atomWithQuery } from 'jotai-tanstack-query';
import { queryClient } from '@/lib/queryClient';

// atomWithQuery per data fetching
export const usersAtom = atomWithQuery(() => ({
  queryKey: ['users'],
  queryFn: async () => {
    const response = await fetch('/api/users');
    return response.json();
  },
  staleTime: 5 * 60 * 1000, // 5 minuti
}));

// atomWithQuery con parametri dinamici
export const userDetailsAtom = atomWithQuery((get) => {
  const selectedUserId = get(selectedUserIdAtom);
  
  return {
    queryKey: ['user', selectedUserId],
    queryFn: async () => {
      if (!selectedUserId) return null;
      const response = await fetch(`/api/users/${selectedUserId}`);
      return response.json();
    },
    enabled: !!selectedUserId, // Query condizionale
  };
});

// Utility atom per prefetching
export const prefetchUserAtom = atom(
  null,
  async (get, set, userId: string) => {
    await queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: async () => {
        const response = await fetch(`/api/users/${userId}`);
        return response.json();
      },
    });
  }
);

// Composizione di query atoms
export const userStatsAtom = atom(async (get) => {
  const users = get(usersAtom);
  
  if (!users.data) return null;
  
  const stats = {
    total: users.data.length,
    active: users.data.filter((u: any) => u.status === 'active').length,
    inactive: users.data.filter((u: any) => u.status === 'inactive').length,
    averageAge: 
      users.data.reduce((sum: number, u: any) => sum + u.age, 0) / 
      users.data.length,
  };
  
  return stats;
});
atomWithStorage (Persist)
typescript
Copia
Scarica
// atoms/persistedAtoms.ts
import { atomWithStorage } from 'jotai/utils';
import { createJSONStorage } from 'jotai/vanilla/utils';

// Storage serializzato con JSON
const storage = createJSONStorage(() => localStorage);

// Atom base con persistenza
export const themeAtom = atomWithStorage<'light' | 'dark' | 'system'>(
  'theme',
  'system',
  storage
);

// Atom complesso con validazione
interface UserPreferences {
  language: string;
  timezone: string;
  notifications: {
    email: boolean;
    push: boolean;
    sms: boolean;
  };
  accessibility: {
    highContrast: boolean;
    reducedMotion: boolean;
    fontSize: number;
  };
}

const defaultPreferences: UserPreferences = {
  language: 'en',
  timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
  notifications: {
    email: true,
    push: false,
    sms: false,
  },
  accessibility: {
    highContrast: false,
    reducedMotion: false,
    fontSize: 16,
  },
};

export const preferencesAtom = atomWithStorage<UserPreferences>(
  'user-preferences',
  defaultPreferences,
  storage,
  {
    getItem: (key, initialValue) => {
      if (typeof window === 'undefined') return initialValue;
      
      const stored = localStorage.getItem(key);
      if (!stored) return initialValue;
      
      try {
        const parsed = JSON.parse(stored);
        // Validazione e merge con defaults
        return {
          ...defaultPreferences,
          ...parsed,
          notifications: {
            ...defaultPreferences.notifications,
            ...parsed.notifications,
          },
          accessibility: {
            ...defaultPreferences.accessibility,
            ...parsed.accessibility,
          },
        };
      } catch {
        return initialValue;
      }
    },
    setItem: (key, value) => {
      localStorage.setItem(key, JSON.stringify(value));
    },
    removeItem: (key) => {
      localStorage.removeItem(key);
    },
  }
);

// Atom derivato da persisted atom
export const notificationCountAtom = atom((get) => {
  const prefs = get(preferencesAtom);
  return Object.values(prefs.notifications).filter(Boolean).length;
});
atomWithReducer per Logica Complessa
typescript
Copia
Scarica
// atoms/shoppingCartAtoms.ts
import { atomWithReducer } from 'jotai/utils';

// Tipi e interfacce
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  sku?: string;
}

interface CartState {
  items: CartItem[];
  lastUpdated: Date | null;
  discountCode?: string;
  subtotal: number;
  tax: number;
  total: number;
}

// Azioni
type CartAction =
  | { type: 'ADD_ITEM'; payload: CartItem }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'APPLY_DISCOUNT'; payload: string }
  | { type: 'CLEAR_CART' }
  | { type: 'LOAD_CART'; payload: CartState };

// Reducer
const cartReducer = (state: CartState, action: CartAction): CartState => {
  const calculateTotals = (items: CartItem[]): CartState => {
    const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const tax = subtotal * 0.1; // 10% tax
    const total = subtotal + tax;
    
    return {
      ...state,
      items,
      subtotal,
      tax,
      total,
      lastUpdated: new Date(),
    };
  };

  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      let newItems: CartItem[];
      if (existingItem) {
        newItems = state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + action.payload.quantity }
            : item
        );
      } else {
        newItems = [...state.items, action.payload];
      }
      
      return calculateTotals(newItems);
    }
    
    case 'REMOVE_ITEM': {
      const newItems = state.items.filter(item => item.id !== action.payload);
      return calculateTotals(newItems);
    }
    
    case 'UPDATE_QUANTITY': {
      const newItems = state.items.map(item =>
        item.id === action.payload.id
          ? { ...item, quantity: Math.max(0, action.payload.quantity) }
          : item
      ).filter(item => item.quantity > 0);
      
      return calculateTotals(newItems);
    }
    
    case 'APPLY_DISCOUNT':
      return {
        ...state,
        discountCode: action.payload,
      };
      
    case 'CLEAR_CART':
      return {
        items: [],
        lastUpdated: new Date(),
        subtotal: 0,
        tax: 0,
        total: 0,
      };
      
    case 'LOAD_CART':
      return action.payload;
      
    default:
      return state;
  }
};

// Initial state
const initialState: CartState = {
  items: [],
  lastUpdated: null,
  subtotal: 0,
  tax: 0,
  total: 0,
};

// Atom con reducer
export const cartAtom = atomWithReducer(initialState, cartReducer);

// Action creators
export const addItem = (item: CartItem) => ({ type: 'ADD_ITEM', payload: item } as const);
export const removeItem = (id: string) => ({ type: 'REMOVE_ITEM', payload: id } as const);
export const updateQuantity = (id: string, quantity: number) => 
  ({ type: 'UPDATE_QUANTITY', payload: { id, quantity } } as const);

// Utility atoms
export const cartItemCountAtom = atom((get) => {
  const cart = get(cartAtom);
  return cart.items.reduce((sum, item) => sum + item.quantity, 0);
});

export const cartTotalAtom = atom((get) => get(cartAtom).total);

export const cartItemsAtom = atom((get) => get(cartAtom).items);
Derived Atoms Chain (3+ Livelli)
typescript
Copia
Scarica
// atoms/dashboardAtoms.ts
import { atom } from 'jotai';

// Livello 1: Data Atoms (fonte primaria)
export const salesDataAtom = atom<Array<{ date: string; amount: number }>>([]);
export const userActivityAtom = atom<Array<{ userId: string; actions: number }>>([]);
export const inventoryAtom = atom<Record<string, { stock: number; threshold: number }>>({});

// Livello 2: Processed Data Atoms
export const monthlySalesAtom = atom((get) => {
  const sales = get(salesDataAtom);
  
  return sales.reduce((acc, sale) => {
    const month = sale.date.substring(0, 7); // YYYY-MM
    acc[month] = (acc[month] || 0) + sale.amount;
    return acc;
  }, {} as Record<string, number>);
});

export const topUsersAtom = atom((get) => {
  const activity = get(userActivityAtom);
  
  return [...activity]
    .sort((a, b) => b.actions - a.actions)
    .slice(0, 10)
    .map(user => ({
      userId: user.userId,
      actions: user.actions,
    }));
});

export const lowStockItemsAtom = atom((get) => {
  const inventory = get(inventoryAtom);
  
  return Object.entries(inventory)
    .filter(([_, item]) => item.stock < item.threshold)
    .map(([id, item]) => ({
      id,
      stock: item.stock,
      threshold: item.threshold,
      needsReorder: item.stock < item.threshold * 0.5,
    }));
});

// Livello 3: Business Metrics Atoms
export const kpiAtom = atom((get) => {
  const monthlySales = get(monthlySalesAtom);
  const topUsers = get(topUsersAtom);
  const lowStockItems = get(lowStockItemsAtom);
  
  const salesValues = Object.values(monthlySales);
  const totalSales = salesValues.reduce((sum, val) => sum + val, 0);
  const averageMonthlySales = salesValues.length > 0 
    ? totalSales / salesValues.length 
    : 0;
  
  const topUserActions = topUsers.reduce((sum, user) => sum + user.actions, 0);
  const criticalItems = lowStockItems.filter(item => item.needsReorder).length;
  
  return {
    totalSales,
    averageMonthlySales,
    topUserActions,
    criticalItems,
    salesGrowth: salesValues.length > 1 
      ? ((salesValues[salesValues.length - 1] - salesValues[0]) / salesValues[0]) * 100
      : 0,
  };
});

// Livello 4: Dashboard State Atom (composizione finale)
export const dashboardStateAtom = atom((get) => {
  const kpi = get(kpiAtom);
  const monthlySales = get(monthlySalesAtom);
  const topUsers = get(topUsersAtom);
  const lowStockItems = get(lowStockItemsAtom);
  
  return {
    metrics: {
      financial: {
        totalSales: kpi.totalSales,
        averageMonthlySales: kpi.averageMonthlySales,
        salesGrowth: kpi.salesGrowth,
      },
      operational: {
        activeUsers: topUsers.length,
        totalUserActions: kpi.topUserActions,
        criticalStockItems: kpi.criticalItems,
      },
    },
    charts: {
      monthlySales: Object.entries(monthlySales).map(([month, amount]) => ({
        month,
        amount,
      })),
      userActivity: topUsers,
    },
    alerts: lowStockItems.map(item => ({
      type: item.needsReorder ? 'critical' : 'warning',
      message: `Low stock: ${item.id} (${item.stock} remaining)`,
    })),
    lastUpdated: new Date().toISOString(),
  };
});

// Livello 5: View-specific Atoms (per componenti UI)
export const dashboardSummaryAtom = atom((get) => {
  const state = get(dashboardStateAtom);
  
  return {
    totalSales: state.metrics.financial.totalSales.toLocaleString('en-US', {
      style: 'currency',
      currency: 'USD',
    }),
    salesGrowth: `${state.metrics.financial.salesGrowth.toFixed(1)}%`,
    activeUsers: state.metrics.operational.activeUsers,
    alertsCount: state.alerts.length,
  };
});
Tabella: 8 Atom Types con Signature TypeScript
Atom Type	Signature TypeScript	Use Case
atom	atom<T>(initialValue: T): Atom<T>	Stato locale semplice, valore iniziale
atomWithStorage	atomWithStorage<T>(key: string, initialValue: T, storage?: Storage): WritableAtom<T>	Persistenza automatica in localStorage
atomFamily	atomFamily<T, P>(initialize: (param: P) => T): (param: P) => Atom<T>	Stato parametrico (items by ID)
atomWithReducer	atomWithReducer<S, A>(initialState: S, reducer: (s: S, a: A) => S): WritableAtom<S, A>	Logica complessa con action pattern
atomWithQuery	atomWithQuery<T>(getOptions: (get: Getter) => QueryOptions<T>): Atom<QueryResult<T>>	Data fetching con TanStack Query
atomWithMutation	atomWithMutation<TData, TVariables>(options: MutationOptions<TData, TVariables>): Atom<MutationResult<TData>>	Mutazioni con caching automatico
atomWithObservable	atomWithObservable<T>(createObservable: (get: Getter) => Observable<T>): Atom<T>	Integrazione con RxJS/streams
focusAtom	focusAtom<T, F>(baseAtom: Atom<T>, focus: (value: T) => F): WritableAtom<F>	Isolamento di sotto-porzioni di stato
11. STATE + NEXT.JS APP ROUTER
Server Components vs Client State Boundary
typescript
Copia
Scarica
// app/layout.tsx - Server Component
import { ReactNode } from 'react';
import { StoreHydration } from '@/components/StoreHydration';
import { ServerDataFetcher } from '@/components/ServerDataFetcher';
import { getServerSession } from 'next-auth';

export default async function RootLayout({
  children,
}: {
  children: ReactNode;
}) {
  // Server-side data fetching
  const session = await getServerSession();
  const userPreferences = await fetchUserPreferences(session?.user?.id);
  const globalConfig = await fetchGlobalConfig();
  
  // Serialize data for client-side hydration
  const initialState = {
    user: session?.user || null,
    preferences: userPreferences,
    config: globalConfig,
  };

  return (
    <html lang="en">
      <body>
        {/* Pass server data to client boundary */}
        <StoreHydration initialState={initialState}>
          <ServerDataFetcher data={initialState}>
            {children}
          </ServerDataFetcher>
        </StoreHydration>
      </body>
    </html>
  );
}

// components/StoreHydration.tsx - Client Component
'use client';
import { ReactNode, useEffect } from 'react';
import { useStore } from '@/store/useStore';
import { useSessionStore } from '@/store/useSessionStore';

interface StoreHydrationProps {
  children: ReactNode;
  initialState: {
    user: any;
    preferences: any;
    config: any;
  };
}

export function StoreHydration({ children, initialState }: StoreHydrationProps) {
  const hydrateUserStore = useSessionStore((state) => state.hydrate);
  const hydrateAppStore = useStore((state) => state.hydrate);
  
  useEffect(() => {
    // Hydrate stores with server data
    hydrateUserStore(initialState.user);
    hydrateAppStore({
      preferences: initialState.preferences,
      config: initialState.config,
    });
  }, [hydrateUserStore, hydrateAppStore, initialState]);

  return <>{children}</>;
}

// components/ServerDataFetcher.tsx - Server Component che passa a Client
import { Suspense } from 'react';
import { ClientDataConsumer } from './ClientDataConsumer';

export async function ServerDataFetcher({
  children,
  data,
}: {
  children: ReactNode;
  data: any;
}) {
  // Simulate async server work
  const additionalData = await fetchAdditionalData();
  
  return (
    <>
      <Suspense fallback={<div>Loading client state...</div>}>
        {/* Client component che riceve props dal server */}
        <ClientDataConsumer serverData={{ ...data, additionalData }}>
          {children}
        </ClientDataConsumer>
      </Suspense>
    </>
  );
}

// components/ClientDataConsumer.tsx - Client Component
'use client';
import { ReactNode, useRef } from 'react';
import { useShallow } from 'zustand/shallow';

interface ClientDataConsumerProps {
  children: ReactNode;
  serverData: any;
}

export function ClientDataConsumer({
  children,
  serverData,
}: ClientDataConsumerProps) {
  const hasHydrated = useRef(false);
  const { hydrate, isHydrated } = useStore(
    useShallow((state) => ({
      hydrate: state.hydrate,
      isHydrated: state.isHydrated,
    }))
  );

  // One-time hydration
  useEffect(() => {
    if (!hasHydrated.current && !isHydrated) {
      hydrate(serverData);
      hasHydrated.current = true;
    }
  }, [hydrate, isHydrated, serverData]);

  // Prevent rendering

ger">Manager</option>
                  <option value="other">Other</option>
                </select>
              )}
            />
            {errors.role && (
              <p className="text-red-500 text-sm mt-1">
                {errors.role.message}
              </p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium mb-1">
              Years of Experience
            </label>
            <Controller
              name="experience"
              control={control}
              render={({ field }) => (
                <div>
                  <input
                    type="range"
                    min="0"
                    max="50"
                    {...field}
                    className="w-full"
                    onChange={(e) => {
                      field.onChange(parseInt(e.target.value));
                      setValue('experience', parseInt(e.target.value), {
                        shouldValidate: true,
                      });
                    }}
                  />
                  <div className="flex justify-between text-sm text-gray-500">
                    <span>0</span>
                    <span>Current: {field.value}</span>
                    <span>50</span>
                  </div>
                </div>
              )}
            />
            {errors.experience && (
              <p className="text-red-500 text-sm mt-1">
                {errors.experience.message}
              </p>
            )}
          </div>
        </div>
      </div>

      {/* Skills Section */}
      <div className="border-b pb-4">
        <h2 className="text-xl font-semibold mb-4">Skills</h2>
        
        <div className="space-y-3">
          {skills.fields.map((field, index) => (
            <div key={field.id} className="flex gap-2 items-center">
              <input
                {...register(`skills.${index}`)}
                type="text"
                className="flex-1 border rounded px-3 py-2"
                placeholder="Enter a skill"
              />

<div className="text-sm text-gray-500 space-y-1">
        {touched && isClientValid && (
          <p>✓ Passed basic validation</p>
        )}
        
        {isValidating && (
          <p>Checking availability...</p>
        )}
        
        {serverValid === true && (
          <p className="text-green-600">✓ Available</p>
        )}
        
        {serverValid === false && errors.length === 0 && (
          <p className="text-red-600">✗ Not available</p>
        )}
      </div>
    </div>
  );
}
File Upload Form State
typescript
Copia
Scarica
// hooks/useFileUpload.ts
'use client';
import { useState, useCallback, ChangeEvent } from 'react';
import { useStore } from '@/store/useFileStore';

interface FileUploadState {
  files: File[];
  previews: string[];
  uploadProgress: Record<string, number>;
  errors: string[];
  isUploading: boolean;
}

interface UseFileUploadReturn {
  state: FileUploadState;
  addFiles: (files: FileList) => void;
  removeFile: (index: number) => void;
  uploadFiles: () => Promise<void>;
  clearAll: () => void;
  validateFiles: () => boolean;
}

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];

export function useFileUpload(): UseFileUploadReturn {
  const [state, set