# CATALOGO-STATE-MANAGEMENT

CATALOGO-STATE-MANAGEMENT per Next.js 14 + React 18+
§ ZUSTAND COMPLETE
Store Creation Patterns
typescript
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

// Basic store pattern
interface BasicStore {
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void
}

export const useBasicStore = create<BasicStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 })
}))

// Factory pattern for reusable stores
interface CounterConfig {
  initialCount?: number
  step?: number
}

const createCounterStore = (config: CounterConfig = {}) => {
  const { initialCount = 0, step = 1 } = config
  
  return create<{
    count: number
    increment: () => void
    decrement: () => void
  }>((set) => ({
    count: initialCount,
    increment: () => set((state) => ({ count: state.count + step })),
    decrement: () => set((state) => ({ count: state.count - step }))
  }))
}

// Type-safe store with selectors
interface UserStore {
  user: User | null
  profile: Profile | null
  isLoading: boolean
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  updateProfile: (updates: Partial<Profile>) => void
}

export const useUserStore = create<UserStore>((set, get) => ({
  user: null,
  profile: null,
  isLoading: false,
  login: async (email: string, password: string) => {
    set({ isLoading: true })
    try {
      const user = await api.login(email, password)
      const profile = await api.getProfile(user.id)
      set({ user, profile, isLoading: false })
    } catch (error) {
      set({ isLoading: false })
      throw error
    }
  },
  logout: () => set({ user: null, profile: null }),
  updateProfile: (updates) => 
    set((state) => ({
      profile: state.profile ? { ...state.profile, ...updates } : null
    }))
}))

// Selector hooks for performance
export const useUser = () => useUserStore((state) => state.user)
export const useProfile = () => useUserStore((state) => state.profile)
export const useIsLoading = () => useUserStore((state) => state.isLoading)
Slices Pattern
typescript
import { StateCreator, create } from 'zustand'
import { devtools } from 'zustand/middleware'

// Define slice types
interface AuthSlice {
  user: User | null
  token: string | null
  login: (credentials: Credentials) => Promise<void>
  logout: () => void
  isAuthenticated: () => boolean
}

interface CartSlice {
  items: CartItem[]
  total: number
  addItem: (item: Product, quantity: number) => void
  removeItem: (productId: string) => void
  clearCart: () => void
  getItemCount: () => number
}

interface UITheme {
  theme: 'light' | 'dark' | 'system'
  toggleTheme: () => void
  setTheme: (theme: 'light' | 'dark' | 'system') => void
}

// Create slice creators
const createAuthSlice: StateCreator<
  AuthSlice & CartSlice & UITheme,
  [],
  [],
  AuthSlice
> = (set, get) => ({
  user: null,
  token: null,
  login: async (credentials) => {
    const response = await api.login(credentials)
    set({ user: response.user, token: response.token })
  },
  logout: () => {
    set({ user: null, token: null })
  },
  isAuthenticated: () => !!get().token
})

const createCartSlice: StateCreator<
  AuthSlice & CartSlice & UITheme,
  [],
  [],
  CartSlice
> = (set, get) => ({
  items: [],
  total: 0,
  addItem: (product, quantity) => {
    set((state) => {
      const existingItemIndex = state.items.findIndex(
        (item) => item.productId === product.id
      )
      
      if (existingItemIndex > -1) {
        const updatedItems = [...state.items]
        updatedItems[existingItemIndex].quantity += quantity
        return {
          items: updatedItems,
          total: calculateTotal(updatedItems)
        }
      }
      
      const newItem: CartItem = {
        productId: product.id,
        name: product.name,
        price: product.price,
        quantity
      }
      
      const newItems = [...state.items, newItem]
      return {
        items: newItems,
        total: calculateTotal(newItems)
      }
    })
  },
  removeItem: (productId) => {
    set((state) => {
      const filteredItems = state.items.filter(
        (item) => item.productId !== productId
      )
      return {
        items: filteredItems,
        total: calculateTotal(filteredItems)
      }
    })
  },
  clearCart: () => {
    set({ items: [], total: 0 })
  },
  getItemCount: () => {
    return get().items.reduce((sum, item) => sum + item.quantity, 0)
  }
})

const createThemeSlice: StateCreator<
  AuthSlice & CartSlice & UITheme,
  [],
  [],
  UITheme
> = (set) => ({
  theme: 'system',
  toggleTheme: () => {
    set((state) => ({
      theme: state.theme === 'light' ? 'dark' : 'light'
    }))
  },
  setTheme: (theme) => {
    set({ theme })
  }
})

// Combine slices into store
export const useAppStore = create<AuthSlice & CartSlice & UITheme>()(
  devtools((...args) => ({
    ...createAuthSlice(...args),
    ...createCartSlice(...args),
    ...createThemeSlice(...args)
  }))
)

// Helper function
const calculateTotal = (items: CartItem[]): number => {
  return items.reduce((total, item) => total + item.price * item.quantity, 0)
}
Persist Middleware
typescript
import { create } from 'zustand'
import { persist, createJSONStorage, PersistOptions } from 'zustand/middleware'
import AsyncStorage from '@react-native-async-storage/async-storage' // For React Native

// Web localStorage persistence
interface PersistedStore {
  settings: Settings
  recentSearches: string[]
  updateSettings: (settings: Partial<Settings>) => void
  addSearch: (query: string) => void
  clearSearches: () => void
}

export const usePersistedStore = create<PersistedStore>()(
  persist(
    (set, get) => ({
      settings: {
        language: 'it',
        notifications: true,
        fontSize: 'medium'
      },
      recentSearches: [],
      updateSettings: (newSettings) => {
        set((state) => ({
          settings: { ...state.settings, ...newSettings }
        }))
      },
      addSearch: (query) => {
        set((state) => {
          const filtered = state.recentSearches.filter((s) => s !== query)
          return {
            recentSearches: [query, ...filtered].slice(0, 10)
          }
        })
      },
      clearSearches: () => {
        set({ recentSearches: [] })
      }
    }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      // Partial persistence
      partialize: (state) => ({
        settings: state.settings,
        recentSearches: state.recentSearches
      }),
      // Migration
      migrate: (persistedState, version) => {
        if (version === 0) {
          // Migrate from v0 to v1
          return {
            ...persistedState,
            settings: {
              ...persistedState.settings,
              // New field added in v1
              darkMode: 'auto'
            }
          }
        }
        return persistedState as PersistedStore
      },
      version: 1
    }
  )
)

// Next.js with hydration
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

// Safe storage for SSR
const createSafeStorage = () => {
  if (typeof window === 'undefined') {
    return {
      getItem: () => null,
      setItem: () => {},
      removeItem: () => {}
    }
  }
  return localStorage
}

interface HydratedStore {
  count: number
  increment: () => void
}

export const useHydratedStore = create<HydratedStore>()(
  persist(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }))
    }),
    {
      name: 'hydrated-store',
      storage: createJSONStorage(() => createSafeStorage())
    }
  )
)

// Custom storage with encryption
const createEncryptedStorage = <T>(key: string) => {
  return {
    getItem: (name: string): T | null => {
      try {
        const item = localStorage.getItem(name)
        if (!item) return null
        // Decrypt here
        const decrypted = decrypt(item, key)
        return JSON.parse(decrypted)
      } catch {
        return null
      }
    },
    setItem: (name: string, value: T) => {
      const encrypted = encrypt(JSON.stringify(value), key)
      localStorage.setItem(name, encrypted)
    },
    removeItem: (name: string) => {
      localStorage.removeItem(name)
    }
  }
}
Devtools Integration
typescript
import { create } from 'zustand'
import { devtools, subscribeWithSelector } from 'zustand/middleware'

// Basic devtools
interface CounterStore {
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void
}

export const useCounterStore = create<CounterStore>()(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), false, 'increment'),
      decrement: () => set((state) => ({ count: state.count - 1 }), false, 'decrement'),
      reset: () => set({ count: 0 }, false, 'reset')
    }),
    {
      name: 'CounterStore',
      enabled: process.env.NODE_ENV === 'development',
      // Store-specific Redux DevTools config
      anonymousActionType: 'CounterStoreAction'
    }
  )
)

// Advanced devtools with actions tracking
interface TodoStore {
  todos: Todo[]
  filter: 'all' | 'active' | 'completed'
  addTodo: (text: string) => void
  toggleTodo: (id: string) => void
  deleteTodo: (id: string) => void
  setFilter: (filter: TodoStore['filter']) => void
}

export const useTodoStore = create<TodoStore>()(
  devtools(
    subscribeWithSelector((set, get) => ({
      todos: [],
      filter: 'all',
      addTodo: (text) => {
        const newTodo: Todo = {
          id: crypto.randomUUID(),
          text,
          completed: false,
          createdAt: new Date().toISOString()
        }
        set(
          (state) => ({ todos: [...state.todos, newTodo] }),
          false,
          { type: 'todos/addTodo', text }
        )
      },
      toggleTodo: (id) => {
        set(
          (state) => ({
            todos: state.todos.map((todo) =>
              todo.id === id ? { ...todo, completed: !todo.completed } : todo
            )
          }),
          false,
          { type: 'todos/toggleTodo', id }
        )
      },
      deleteTodo: (id) => {
        set(
          (state) => ({
            todos: state.todos.filter((todo) => todo.id !== id)
          }),
          false,
          { type: 'todos/deleteTodo', id }
        )
      },
      setFilter: (filter) => {
        set({ filter }, false, { type: 'todos/setFilter', filter })
      }
    })),
    {
      name: 'TodoStore',
      store: 'todoStore',
      trace: true,
      traceLimit: 25
    }
  )
)

// Custom middleware with devtools
const withLogger = <T extends object>(
  config: StateCreator<T, [], []>
): StateCreator<T, [], []> => {
  return (set, get, api) => {
    const loggedSet: typeof set = (...args) => {
      const [partial, replace, actionName] = args
      console.groupCollapsed(
        `%cZustand Action: %c${actionName || 'unknown'}`,
        'color: #9e9e9e; font-weight: normal;',
        'color: #007acc; font-weight: bold;'
      )
      console.log('Previous State:', get())
      console.log('Action:', { partial, replace, actionName })
      set(...args)
      console.log('Next State:', get())
      console.groupEnd()
      return
    }
    
    return config(loggedSet, get, api)
  }
}

// Combined middleware stack
export const useAdvancedStore = create<CounterStore>()(
  devtools(
    withLogger((set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), false, 'increment'),
      decrement: () => set((state) => ({ count: state.count - 1 }), false, 'decrement'),
      reset: () => set({ count: 0 }, false, 'reset')
    })),
    { name: 'AdvancedStore' }
  )
)
Immer Middleware
typescript
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

// Basic immer usage
interface ImmerStore {
  items: Item[]
  selectedIds: Set<string>
  addItem: (item: Item) => void
  removeItem: (id: string) => void
  toggleSelection: (id: string) => void
  updateItem: (id: string, updates: Partial<Item>) => void
  clearSelection: () => void
}

export const useImmerStore = create<ImmerStore>()(
  immer((set) => ({
    items: [],
    selectedIds: new Set(),
    addItem: (item) => {
      set((state) => {
        state.items.push(item)
      })
    },
    removeItem: (id) => {
      set((state) => {
        state.items = state.items.filter((item) => item.id !== id)
        state.selectedIds.delete(id)
      })
    },
    toggleSelection: (id) => {
      set((state) => {
        if (state.selectedIds.has(id)) {
          state.selectedIds.delete(id)
        } else {
          state.selectedIds.add(id)
        }
      })
    },
    updateItem: (id, updates) => {
      set((state) => {
        const item = state.items.find((item) => item.id === id)
        if (item) {
          Object.assign(item, updates)
        }
      })
    },
    clearSelection: () => {
      set((state) => {
        state.selectedIds.clear()
      })
    }
  }))
)

// Nested updates with immer
interface NestedStore {
  user: {
    profile: {
      name: string
      address: {
        street: string
        city: string
        country: string
      }
      preferences: {
        theme: 'light' | 'dark'
        notifications: boolean
        language: string
      }
    }
    documents: Array<{
      id: string
      name: string
      status: 'pending' | 'approved' | 'rejected'
    }>
  }
  updateAddress: (address: Partial<NestedStore['user']['profile']['address']>) => void
  updatePreference: <K extends keyof NestedStore['user']['profile']['preferences']>(
    key: K,
    value: NestedStore['user']['profile']['preferences'][K]
  ) => void
  addDocument: (document: NestedStore['user']['documents'][0]) => void
  updateDocumentStatus: (id: string, status: NestedStore['user']['documents'][0]['status']) => void
}

export const useNestedStore = create<NestedStore>()(
  immer((set) => ({
    user: {
      profile: {
        name: '',
        address: {
          street: '',
          city: '',
          country: ''
        },
        preferences: {
          theme: 'light',
          notifications: true,
          language: 'en'
        }
      },
      documents: []
    },
    updateAddress: (address) => {
      set((state) => {
        Object.assign(state.user.profile.address, address)
      })
    },
    updatePreference: (key, value) => {
      set((state) => {
        state.user.profile.preferences[key] = value
      })
    },
    addDocument: (document) => {
      set((state) => {
        state.user.documents.push(document)
      })
    },
    updateDocumentStatus: (id, status) => {
      set((state) => {
        const doc = state.user.documents.find((d) => d.id === id)
        if (doc) {
          doc.status = status
        }
      })
    }
  }))
)

// Combine immer with other middleware
export const useCompleteStore = create<ImmerStore>()(
  devtools(
    persist(
      immer((set) => ({
        items: [],
        selectedIds: new Set(),
        addItem: (item) => {
          set((state) => {
            state.items.push(item)
          })
        },
        removeItem: (id) => {
          set((state) => {
            state.items = state.items.filter((item) => item.id !== id)
            state.selectedIds.delete(id)
          })
        },
        toggleSelection: (id) => {
          set((state) => {
            if (state.selectedIds.has(id)) {
              state.selectedIds.delete(id)
            } else {
              state.selectedIds.add(id)
            }
          })
        },
        updateItem: (id, updates) => {
          set((state) => {
            const item = state.items.find((item) => item.id === id)
            if (item) {
              Object.assign(item, updates)
            }
          })
        },
        clearSelection: () => {
          set((state) => {
            state.selectedIds.clear()
          })
        }
      })),
      {
        name: 'complete-store',
        storage: createJSONStorage(() => localStorage)
      }
    ),
    { name: 'CompleteStore' }
  )
)
Computed Values
typescript
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'
import { shallow } from 'zustand/shallow'

// Inline computed values
interface CartStore {
  items: CartItem[]
  taxRate: number
  // Computed getters
  get subtotal(): number
  get tax(): number
  get total(): number
  get itemCount(): number
  get isEmpty(): boolean
  // Actions
  addItem: (item: Omit<CartItem, 'id'>) => void
  removeItem: (id: string) => void
  updateQuantity: (id: string, quantity: number) => void
}

export const useCartStore = create<CartStore>()(
  devtools(
    (set, get) => ({
      items: [],
      taxRate: 0.22,
      
      // Computed properties
      get subtotal() {
        return get().items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        )
      },
      
      get tax() {
        return get().subtotal * get().taxRate
      },
      
      get total() {
        return get().subtotal + get().tax
      },
      
      get itemCount() {
        return get().items.reduce((sum, item) => sum + item.quantity, 0)
      },
      
      get isEmpty() {
        return get().items.length === 0
      },
      
      // Actions
      addItem: (newItem) => {
        const item: CartItem = {
          ...newItem,
          id: crypto.randomUUID()
        }
        set((state) => ({ items: [...state.items, item] }))
      },
      
      removeItem: (id) => {
        set((state) => ({
          items: state.items.filter((item) => item.id !== id)
        }))
      },
      
      updateQuantity: (id, quantity) => {
        set((state) => ({
          items: state.items.map((item) =>
            item.id === id ? { ...item, quantity } : item
          )
        }))
      }
    }),
    { name: 'CartStore' }
  )
)

// Derived store pattern
const createDerivedStore = <T, R>(
  baseStore: T,
  derive: (state: T) => R
) => {
  return derive(baseStore)
}

// Example usage with memoization
import { useMemo } from 'react'

interface ProductStore {
  products: Product[]
  searchQuery: string
  categoryFilter: string[]
  priceRange: [number, number]
  sortBy: 'name' | 'price' | 'rating'
  sortOrder: 'asc' | 'desc'
}

export const useProductStore = create<ProductStore>()(
  devtools((set) => ({
    products: [],
    searchQuery: '',
    categoryFilter: [],
    priceRange: [0, 1000],
    sortBy: 'name',
    sortOrder: 'asc'
  }))
)

// Custom hook for computed values with memoization
export const useFilteredProducts = () => {
  const { products, searchQuery, categoryFilter, priceRange, sortBy, sortOrder } =
    useProductStore()
    
  return useMemo(() => {
    let filtered = products
    
    // Apply search
    if (searchQuery) {
      const query = searchQuery.toLowerCase()
      filtered = filtered.filter(
        (product) =>
          product.name.toLowerCase().includes(query) ||
          product.description.toLowerCase().includes(query)
      )
    }
    
    // Apply category filter
    if (categoryFilter.length > 0) {
      filtered = filtered.filter((product) =>
        categoryFilter.includes(product.category)
      )
    }
    
    // Apply price range
    filtered = filtered.filter(
      (product) =>
        product.price >= priceRange[0] && product.price <= priceRange[1]
    )
    
    // Apply sorting
    filtered = [...filtered].sort((a, b) => {
      let aValue: any
      let bValue: any
      
      switch (sortBy) {
        case 'price':
          aValue = a.price
          bValue = b.price
          break
        case 'rating':
          aValue = a.rating
          bValue = b.rating
          break
        default:
          aValue = a.name
          bValue = b.name
      }
      
      if (sortOrder === 'asc') {
        return aValue > bValue ? 1 : -1
      } else {
        return aValue < bValue ? 1 : -1
      }
    })
    
    return filtered
  }, [products, searchQuery, categoryFilter, priceRange, sortBy, sortOrder])
}

// Computed store with dependencies
interface DashboardStore {
  sales: Sale[]
  expenses: Expense[]
  get metrics(): {
    totalRevenue: number
    totalExpenses: number
    netProfit: number
    profitMargin: number
    topProducts: Array<{ product: string; revenue: number }>
  }
}

export const useDashboardStore = create<DashboardStore>()(
  devtools((set, get) => ({
    sales: [],
    expenses: [],
    get metrics() {
      const { sales, expenses } = get()
      
      const totalRevenue = sales.reduce(
        (sum, sale) => sum + sale.amount,
        0
      )
      
      const totalExpenses = expenses.reduce(
        (sum, expense) => sum + expense.amount,
        0
      )
      
      const netProfit = totalRevenue - totalExpenses
      const profitMargin = totalRevenue > 0 ? (netProfit / totalRevenue) * 100 : 0
      
      // Calculate top products
      const productRevenue = sales.reduce((acc, sale) => {
        acc[sale.productId] = (acc[sale.productId] || 0) + sale.amount
        return acc
      }, {} as Record<string, number>)
      
      const topProducts = Object.entries(productRevenue)
        .sort(([, a], [, b]) => b - a)
        .slice(0, 5)
        .map(([productId, revenue]) => ({
          product: productId,
          revenue
        }))
      
      return {
        totalRevenue,
        totalExpenses,
        netProfit,
        profitMargin,
        topProducts
      }
    }
  }))
)
Actions Organization
typescript
import { create } from 'zustand'
import { immer } from 'zustand/middleware/immer'

// Action creators pattern
interface UserActions {
  login: (credentials: Credentials) => Promise<void>
  logout: () => void
  updateProfile: (updates: Partial<Profile>) => Promise<void>
  changePassword: (oldPassword: string, newPassword: string) => Promise<void>
}

interface UserState {
  user: User | null
  profile: Profile | null
  isLoading: boolean
  error: string | null
}

const createUserActions = (
  set: (fn: (state: UserState & UserActions) => void) => void,
  get: () => UserState & UserActions
): UserActions => ({
  login: async (credentials) => {
    set((state) => {
      state.isLoading = true
      state.error = null
    })
    
    try {
      const response = await api.login(credentials)
      set((state) => {
        state.user = response.user
        state.profile = response.profile
        state.isLoading = false
      })
    } catch (error) {
      set((state) => {
        state.isLoading = false
        state.error = error instanceof Error ? error.message : 'Login failed'
      })
    }
  },
  
  logout: () => {
    set((state) => {
      state.user = null
      state.profile = null
    })
  },
  
  updateProfile: async (updates) => {
    const { user } = get()
    if (!user) throw new Error('User not authenticated')
    
    set((state) => {
      state.isLoading = true
      state.error = null
    })
    
    try {
      const updatedProfile = await api.updateProfile(user.id, updates)
      set((state) => {
        state.profile = updatedProfile
        state.isLoading = false
      })
    } catch (error) {
      set((state) => {
        state.isLoading = false
        state.error = error instanceof Error ? error.message : 'Update failed'
      })
    }
  },
  
  changePassword: async (oldPassword, newPassword) => {
    const { user } = get()
    if (!user) throw new Error('User not authenticated')
    
    set((state) => {
      state.isLoading = true
      state.error = null
    })
    
    try {
      await api.changePassword(user.id, oldPassword, newPassword)
      set((state) => {
        state.isLoading = false
      })
    } catch (error) {
      set((state) => {
        state.isLoading = false
        state.error = error instanceof Error ? error.message : 'Password change failed'
      })
    }
  }
})

export const useUserStore = create<UserState & UserActions>()(
  immer((set, get) => ({
    user: null,
    profile: null,
    isLoading: false,
    error: null,
    ...createUserActions(set, get as () => UserState & UserActions)
  }))
)

// Command pattern for undo/redo
interface Command<T> {
  execute: () => void
  undo: () => void
  description: string
}

interface CommandStore<T> {
  history: Array<Command<T>>
  future: Array<Command<T>>
  currentState: T
  execute: (command: Command<T>) => void
  undo: () => void
  redo: () => void
  canUndo: boolean
  canRedo: boolean
  clearHistory: () => void
}

const createCommandStore = <T>(initialState: T) => {
  return create<CommandStore<T>>((set, get) => ({
    history: [],
    future: [],
    currentState: initialState,
    
    execute: (command) => {
      command.execute()
      set((state) => ({
        history: [...state.history, command],
        future: [], // Clear future when new command executed
        currentState: { ...state.currentState } // Trigger update
      }))
    },
    
    undo: () => {
      const { history, future } = get()
      if (history.length === 0) return
      
      const lastCommand = history[history.length - 1]
      lastCommand.undo()
      
      set((state) => ({
        history: state.history.slice(0, -1),
        future: [lastCommand, ...state.future]
      }))
    },
    
    redo: () => {
      const { future } = get()
      if (future.length === 0) return
      
      const nextCommand = future[0]
      nextCommand.execute()
      
      set((state) => ({
        history: [...state.history, nextCommand],
        future: state.future.slice(1)
      }))
    },
    
    get canUndo() {
      return get().history.length > 0
    },
    
    get canRedo() {
      return get().future.length > 0
    },
    
    clearHistory: () => {
      set({ history: [], future: [] })
    }
  }))
}

// Thunk pattern for async actions
type ThunkAction<T, R = void> = (
  dispatch: (action: (state: T) => void) => void,
  getState: () => T
) => Promise<R> | R

interface ThunkStore<T> {
  state: T
  dispatch: (action: (state: T) => void) => void
  runThunk: <R>(thunk: ThunkAction<T, R>) => Promise<R>
}

const createThunkStore = <T>(initialState: T) => {
  return create<ThunkStore<T>>((set, get) => ({
    state: initialState,
    
    dispatch: (action) => {
      set((store) => ({
        state: action(store.state)
      }))
    },
    
    runThunk: async (thunk) => {
      return await thunk(get().dispatch, () => get().state)
    }
  }))
}

// Module pattern for large stores
export const createAuthModule = () => {
  const useAuthStore = create<{
    user: User | null
    token: string | null
    login: (email: string, password: string) => Promise<void>
    logout: () => void
    register: (userData: RegisterData) => Promise<void>
  }>((set) => ({
    user: null,
    token: null,
    login: async (email, password) => {
      // Implementation
    },
    logout: () => {
      // Implementation
    },
    register: async (userData) => {
      // Implementation
    }
  }))
  
  return {
    useAuthStore,
    // Export selectors
    useUser: () => useAuthStore((state) => state.user),
    useIsAuthenticated: () => useAuthStore((state) => !!state.token)
  }
}

export const createCartModule = () => {
  const useCartStore = create<{
    items: CartItem[]
    addItem: (item: Product) => void
    removeItem: (id: string) => void
    clearCart: () => void
  }>((set) => ({
    items: [],
    addItem: (item) => {
      // Implementation
    },
    removeItem: (id) => {
      // Implementation
    },
    clearCart: () => {
      // Implementation
    }
  }))
  
  return {
    useCartStore,
    // Export computed values
    useCartTotal: () => useCartStore((state) => 
      state.items.reduce((total, item) => total + item.price, 0)
    ),
    useItemCount: () => useCartStore((state) => state.items.length)
  }
}

// Combine modules
export const appModules = {
  auth: createAuthModule(),
  cart: createCartModule()
}
§ JOTAI
Atoms Creation
typescript
import { atom } from 'jotai'
import { atomWithStorage } from 'jotai/utils'

// Basic atoms
export const countAtom = atom(0)
export const textAtom = atom('')
export const arrayAtom = atom<string[]>([])
export const objectAtom = atom<{ [key: string]: any }>({})

// Read-only atoms
export const readOnlyAtom = atom((get) => {
  const count = get(countAtom)
  const text = get(textAtom)
  return `${text}: ${count}`
})

// Write-only atoms (actions)
export const incrementAtom = atom(
  null, // read function - we don't need to read anything
  (get, set) => {
    set(countAtom, get(countAtom) + 1)
  }
)

export const addItemAtom = atom(
  null,
  (get, set, item: string) => {
    const currentArray = get(arrayAtom)
    set(arrayAtom, [...currentArray, item])
  }
)

// Atom with localStorage persistence
export const themeAtom = atomWithStorage<'light' | 'dark'>('theme', 'light')
export const userPreferencesAtom = atomWithStorage<UserPreferences>(
  'userPreferences',
  {
    language: 'en',
    notifications: true,
    fontSize: 'medium'
  }
)

// Async atoms
export const userAtom = atom(async () => {
  const response = await fetch('/api/user')
  const data = await response.json()
  return data as User
})

// Atom with validation
export const emailAtom = atom(
  '',
  (get, set, newEmail: string) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (emailRegex.test(newEmail)) {
      set(emailAtom, newEmail)
    } else {
      throw new Error('Invalid email address')
    }
  }
)

// Atom families
import { atomFamily } from 'jotai/utils'

export const todoAtomFamily = atomFamily((id: string) => 
  atom<Todo>({ id, text: '', completed: false })
)

// Custom atom with side effects
export const counterWithEffectAtom = atom(
  (get) => get(countAtom),
  (get, set, newValue: number) => {
    // Perform side effects
    console.log(`Changing counter from ${get(countAtom)} to ${newValue}`)
    
    // Update the atom
    set(countAtom, newValue)
    
    // More side effects
    if (newValue > 10) {
      console.warn('Counter value is getting high!')
    }
  }
)

// Atom with reset functionality
export const resettableAtom = atom(
  (get) => get(countAtom),
  (get, set, newValue: number | 'reset') => {
    if (newValue === 'reset') {
      set(countAtom, 0)
    } else {
      set(countAtom, newValue)
    }
  }
)
Derived Atoms
typescript
import { atom } from 'jotai'

// Basic derived atoms
export const priceAtom = atom(100)
export const quantityAtom = atom(1)
export const discountAtom = atom(10) // percentage

export const subtotalAtom = atom((get) => {
  const price = get(priceAtom)
  const quantity = get(quantityAtom)
  return price * quantity
})

export const discountAmountAtom = atom((get) => {
  const subtotal = get(subtotalAtom)
  const discount = get(discountAtom)
  return (subtotal * discount) / 100
})

export const totalAtom = atom((get) => {
  const subtotal = get(subtotalAtom)
  const discount = get(discountAmountAtom)

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-32-STATE-MANAGEMENT
Prompt ID: 32 / 48
Parte: 2
Exported: 2026-02-06T12:43:03.595Z
Characters: 626
════════════════════════════════════════════════════════════

bles.updates } : undefined
        )
      }
      
      if (previousPosts) {
        queryClient.setQueryData<Post[]>(['posts'], (old) =>
          old?.map((post) =>
            post.id === variables.id ? { ...post, ...variables.updates } : post
          )
        )
      }
      
      // Return context for rollback
      return { previousPost, previousPosts }
    },
    onError: (error, variables, context) => {
      // Rollback on error
      if (context?.previousPost) {
        queryClient.setQueryData(['posts', variables.id], context.previousPost)
      }
      if (context?.previousPosts) {
        queryClient