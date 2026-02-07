# CATALOGO UI/UX - COMPONENT PATTERNS REACT v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Pattern architetturali React, composizione componenti, gestione stato locale

---

## PATTERN 1: Compound Components

**Metadata:**

| Campo         | Valore                                          |
| ------------- | ----------------------------------------------- |
| Categoria     | Structural                                      |
| Complessità   | Medium                                          |
| React Version | 18+                                             |
| Use Case      | Componenti con relazione padre-figlio implicita |

**Implementazione:**

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface TabsContextType {
  active: number;
  setActive: (index: number) => void;
}

const TabsContext = createContext<TabsContextType | null>(null);

const useTabs = () => {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error('Tabs must be used within TabsProvider');
  return ctx;
};

export const Tabs: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [active, setActive] = useState(0);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
};

export const TabList: React.FC<{ children: ReactNode }> = ({ children }) => {
  return <div role="tablist">{children}</div>;
};

export const Tab: React.FC<{ index: number; children: ReactNode }> = ({ index, children }) => {
  const { active, setActive } = useTabs();
  return (
    <button role="tab" aria-selected={active === index} onClick={() => setActive(index)}>
      {children}
    </button>
  );
};

export const TabPanel: React.FC<{ index: number; children: ReactNode }> = ({ index, children }) => {
  const { active } = useTabs();
  return active === index ? <div role="tabpanel">{children}</div> : null;
};
```

**Quando Usare:** ✅ UI composte | ✅ API dichiarativa | ✅ No prop drilling

**Quando Evitare:** ❌ Componenti semplici | ❌ Performance critiche

---

## PATTERN 2: Render Props

**Metadata:**

| Campo         | Valore                                        |
| ------------- | --------------------------------------------- |
| Categoria     | Behavioral                                    |
| Complessità   | Medium                                        |
| React Version | 18+                                           |
| Use Case      | Condivisione logica tramite funzione children |

**Implementazione:**

```typescript
import React, { useState, ReactNode } from 'react';

interface MouseProps {
  children: (state: { x: number; y: number }) => ReactNode;
}

export const MouseTracker: React.FC<MouseProps> = ({ children }) => {
  const [pos, setPos] = useState({ x: 0, y: 0 });

  const handleMove = (e: React.MouseEvent<HTMLDivElement>) => {
    setPos({ x: e.clientX, y: e.clientY });
  };

  return (
    <div style={{ height: '200px', border: '1px solid black' }} onMouseMove={handleMove}>
      {children(pos)}
    </div>
  );
};
```

**Quando Usare:** ✅ Riutilizzo logica | ✅ Separazione logica/UI | ✅ API flessibile

**Quando Evitare:** ❌ Nesting profondo | ❌ Hook più semplici disponibili

---

## PATTERN 3: Custom Hooks

**Metadata:**

| Campo         | Valore                     |
| ------------- | -------------------------- |
| Categoria     | Behavioral                 |
| Complessità   | Low                        |
| React Version | 18+                        |
| Use Case      | Riutilizzo logica stateful |

**Implementazione:**

```typescript
import { useEffect, useState } from 'react';

export const useOnlineStatus = (): boolean => {
  const [online, setOnline] = useState<boolean>(navigator.onLine);

  useEffect(() => {
    const on = () => setOnline(true);
    const off = () => setOnline(false);
    window.addEventListener('online', on);
    window.addEventListener('offline', off);
    return () => {
      window.removeEventListener('online', on);
      window.removeEventListener('offline', off);
    };
  }, []);

  return online;
};

export const StatusIndicator: React.FC = () => {
  const online = useOnlineStatus();
  return <span>{online ? 'Online' : 'Offline'}</span>;
};
```

**Quando Usare:** ✅ Condivisione logica | ✅ Composizione funzionale | ✅ API semplice

**Quando Evitare:** ❌ Dipendenze complesse | ❌ Side effects non controllati

---

## PATTERN 4: Higher-Order Components (HOC)

**Metadata:**

| Campo         | Valore                              |
| ------------- | ----------------------------------- |
| Categoria     | Structural                          |
| Complessità   | Medium                              |
| React Version | 18+                                 |
| Use Case      | Estensione comportamento componenti |

**Implementazione:**

```typescript
import React, { ComponentType } from 'react';

interface WithLoadingProps {
  loading: boolean;
}

export const withLoading = <P extends object>(
  Wrapped: ComponentType<P>
): React.FC<P & WithLoadingProps> => {
  return ({ loading, ...props }) => {
    if (loading) return <div>Loading...</div>;
    return <Wrapped {...(props as P)} />;
  };
};

interface DataProps {
  data: string;
}

export const DataView: React.FC<DataProps> = ({ data }) => <div>{data}</div>;
export const DataViewWithLoading = withLoading(DataView);
```

**Quando Usare:** ✅ Cross-cutting concerns | ✅ Logging, auth, loading

**Quando Evitare:** ❌ Hook disponibili | ❌ Debug complesso

---

## PATTERN 5: Provider Pattern

**Metadata:**

| Campo         | Valore                        |
| ------------- | ----------------------------- |
| Categoria     | Structural                    |
| Complessità   | Medium                        |
| React Version | 18+                           |
| Use Case      | Stato globale tramite Context |

**Implementazione:**

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggle: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

export const ThemeProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const toggle = () => setTheme((t) => (t === 'light' ? 'dark' : 'light'));
  return <ThemeContext.Provider value={{ theme, toggle }}>{children}</ThemeContext.Provider>;
};

export const useTheme = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
};
```

**Quando Usare:** ✅ Stato condiviso | ✅ Configurazioni globali | ✅ Riduzione prop drilling

**Quando Evitare:** ❌ Stato locale semplice | ❌ Aggiornamenti frequenti

---

## PATTERN 6: Container/Presentational

**Metadata:**

| Campo         | Valore                  |
| ------------- | ----------------------- |
| Categoria     | Architectural           |
| Complessità   | Low                     |
| React Version | 18+                     |
| Use Case      | Separazione logica e UI |

**Implementazione:**

```typescript
import React, { useEffect, useState } from 'react';

interface User {
  id: number;
  name: string;
}

// Presentational Component
export const UserList: React.FC<{ users: User[] }> = ({ users }) => (
  <ul>
    {users.map((u) => <li key={u.id}>{u.name}</li>)}
  </ul>
);

// Container Component
export const UserContainer: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    setUsers([{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]);
  }, []);

  return <UserList users={users} />;
};
```

**Quando Usare:** ✅ Separazione responsabilità | ✅ Testabilità | ✅ Manutenibilità

---

## PATTERN 7: Controlled vs Uncontrolled

**Metadata:**

| Campo         | Valore                     |
| ------------- | -------------------------- |
| Categoria     | Behavioral                 |
| Complessità   | Low                        |
| React Version | 18+                        |
| Use Case      | Gestione input controllati |

**Implementazione:**

```typescript
import React, { useRef, useState } from 'react';

// Controlled
export const ControlledInput: React.FC = () => {
  const [value, setValue] = useState<string>('');
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Uncontrolled
export const UncontrolledInput: React.FC = () => {
  const ref = useRef<HTMLInputElement>(null);
  const handleClick = () => {
    if (ref.current) alert(ref.current.value);
  };
  return (
    <div>
      <input ref={ref} defaultValue="Hello" />
      <button onClick={handleClick}>Show</button>
    </div>
  );
};
```

**Quando Usare:** ✅ Controlled: validazione | ✅ Uncontrolled: performance

---

## PATTERN 8: State Reducer

**Metadata:**

| Campo         | Valore                            |
| ------------- | --------------------------------- |
| Categoria     | Behavioral                        |
| Complessità   | Medium                            |
| React Version | 18+                               |
| Use Case      | Estensione comportamento stateful |

**Implementazione:**

```typescript
import React, { useReducer } from 'react';

type Action = { type: 'inc' } | { type: 'dec' };

const reducer = (state: number, action: Action): number => {
  switch (action.type) {
    case 'inc': return state + 1;
    case 'dec': return state - 1;
    default: return state;
  }
};

export const Counter: React.FC = () => {
  const [count, dispatch] = useReducer(reducer, 0);
  return (
    <div>
      <button onClick={() => dispatch({ type: 'dec' })}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'inc' })}>+</button>
    </div>
  );
};
```

**Quando Usare:** ✅ Logica complessa | ✅ Stato prevedibile | ✅ Estendibilità

---

## PATTERN 9: Props Getters

**Metadata:**

| Campo         | Valore                        |
| ------------- | ----------------------------- |
| Categoria     | API Design                    |
| Complessità   | Medium                        |
| React Version | 18+                           |
| Use Case      | API flessibile per componenti |

**Implementazione:**

```typescript
import React, { useState } from 'react';

export const useToggle = () => {
  const [on, setOn] = useState(false);
  const toggle = () => setOn((v) => !v);

  const getTogglerProps = (props: React.ButtonHTMLAttributes<HTMLButtonElement> = {}) => ({
    ...props,
    onClick: (e: React.MouseEvent<HTMLButtonElement>) => {
      props.onClick?.(e);
      toggle();
    },
  });

  return { on, getTogglerProps };
};

export const ToggleButton: React.FC = () => {
  const { on, getTogglerProps } = useToggle();
  return <button {...getTogglerProps()}>{on ? 'ON' : 'OFF'}</button>;
};
```

---

## PATTERN 10: Compound Component with Context

**Metadata:**

| Campo         | Valore                                  |
| ------------- | --------------------------------------- |
| Categoria     | Structural                              |
| Complessità   | High                                    |
| React Version | 18+                                     |
| Use Case      | Compound components con stato condiviso |

**Implementazione:**

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface DropdownContextType {
  open: boolean;
  toggle: () => void;
}

const DropdownContext = createContext<DropdownContextType | null>(null);

const useDropdown = () => {
  const ctx = useContext(DropdownContext);
  if (!ctx) throw new Error('Dropdown must be used within provider');
  return ctx;
};

export const Dropdown: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [open, setOpen] = useState(false);
  const toggle = () => setOpen((v) => !v);
  return <DropdownContext.Provider value={{ open, toggle }}>{children}</DropdownContext.Provider>;
};

export const DropdownTrigger: React.FC<{ children: ReactNode }> = ({ children }) => {
  const { toggle } = useDropdown();
  return <button onClick={toggle}>{children}</button>;
};

export const DropdownMenu: React.FC<{ children: ReactNode }> = ({ children }) => {
  const { open } = useDropdown();
  return open ? <div>{children}</div> : null;
};
```

---

## PATTERN 11: Slot Pattern

**Implementazione:**

```typescript
import React, { ReactNode } from 'react';

interface CardProps {
  header?: ReactNode;
  footer?: ReactNode;
  children: ReactNode;
}

export const Card: React.FC<CardProps> = ({ header, footer, children }) => (
  <div className="border rounded p-4 space-y-2">
    {header && <div className="font-bold">{header}</div>}
    <div className="text-sm">{children}</div>
    {footer && <div className="text-xs text-gray-500">{footer}</div>}
  </div>
);
```

---

## PATTERN 12: Polymorphic Components

**Implementazione:**

```typescript
import React, { ElementType, ComponentPropsWithoutRef } from 'react';

type PolymorphicProps<T extends ElementType> = {
  as?: T;
  children: React.ReactNode;
} & ComponentPropsWithoutRef<T>;

export const Text = <T extends ElementType = 'span'>({
  as,
  children,
  ...props
}: PolymorphicProps<T>) => {
  const Component = as || 'span';
  return <Component {...props}>{children}</Component>;
};
```

---

## PATTERN 13: Headless Components

**Implementazione:**

```typescript
import React, { useState, ReactNode } from 'react';

interface ToggleProps {
  children: (state: { on: boolean; toggle: () => void }) => ReactNode;
}

export const ToggleHeadless: React.FC<ToggleProps> = ({ children }) => {
  const [on, setOn] = useState(false);
  const toggle = () => setOn((v) => !v);
  return <>{children({ on, toggle })}</>;
};
```

---

## PATTERN 14: Forwarding Refs

**Implementazione:**

```typescript
import React, { forwardRef, InputHTMLAttributes } from 'react';

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, ...props }, ref) => (
    <label className="flex flex-col gap-1">
      <span>{label}</span>
      <input ref={ref} {...props} className="border p-1" />
    </label>
  )
);
```

---

## PATTERN 15: Error Boundary

**Implementazione:**

```typescript
import React, { Component, ReactNode } from 'react';

interface ErrorBoundaryState {
  hasError: boolean;
}

export class ErrorBoundary extends Component<{ children: ReactNode }, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false };

  static getDerivedStateFromError(): ErrorBoundaryState {
    return { hasError: true };
  }

  componentDidCatch(error: Error) {
    console.error('ErrorBoundary caught:', error);
  }

  render() {
    if (this.state.hasError) return <div>Something went wrong.</div>;
    return this.props.children;
  }
}
```

---

## PATTERN 16: Suspense Pattern

**Implementazione:**

```typescript
import React, { Suspense, lazy } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

export const AppSuspense: React.FC = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <LazyComponent />
  </Suspense>
);
```

---

## PATTERN 17: Optimistic Updates

**Implementazione:**

```typescript
import React, { useState } from 'react';

export const OptimisticList: React.FC = () => {
  const [items, setItems] = useState<string[]>(['A', 'B']);

  const addItem = async (value: string) => {
    setItems((prev) => [...prev, value]); // Optimistic
    try {
      await new Promise((r) => setTimeout(r, 500)); // API call
    } catch {
      setItems((prev) => prev.filter((i) => i !== value)); // Rollback
    }
  };

  return (
    <div>
      <button onClick={() => addItem('C')}>Add</button>
      <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>
    </div>
  );
};
```

---

## PATTERN 18: Derived State

**Implementazione:**

```typescript
import React, { useMemo } from 'react';

export const SumView: React.FC<{ numbers: number[] }> = ({ numbers }) => {
  const sum = useMemo(() => numbers.reduce((a, b) => a + b, 0), [numbers]);
  return (
    <div>
      <p>Numbers: {numbers.join(', ')}</p>
      <p>Sum: {sum}</p>
    </div>
  );
};
```

---

## PATTERN 19: State Machine (XState)

**Implementazione:**

```typescript
import React from 'react';
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'off',
  states: {
    off: { on: { TOGGLE: 'on' } },
    on: { on: { TOGGLE: 'off' } },
  },
});

export const ToggleMachine: React.FC = () => {
  const [state, send] = useMachine(toggleMachine);
  return (
    <button onClick={() => send('TOGGLE')}>
      {state.matches('on') ? 'ON' : 'OFF'}
    </button>
  );
};
```

---

## PATTERN 20: Composition over Inheritance

**Implementazione:**

```typescript
import React, { ReactNode } from 'react';

export const Box: React.FC<{ children: ReactNode }> = ({ children }) => (
  <div className="p-4 border">{children}</div>
);

export const Alert: React.FC<{ message: string }> = ({ message }) => (
  <Box>
    <strong>Alert:</strong> {message}
  </Box>
);
```

---

## PATTERN DECISION MATRIX

| Pattern                  | Complessità | Use Case Principale            | Alternative       |
| ------------------------ | ----------- | ------------------------------ | ----------------- |
| Compound Components      | Medium      | UI composte correlate          | Render Props      |
| Render Props             | Medium      | Condivisione logica            | Custom Hooks      |
| Custom Hooks             | Low         | Riutilizzo logica stateful     | HOC               |
| HOC                      | Medium      | Cross-cutting concerns         | Hooks             |
| Provider Pattern         | Medium      | Stato globale                  | Redux, Zustand    |
| Container/Presentational | Low         | Separazione responsabilità     | Hooks             |
| Controlled/Uncontrolled  | Low         | Gestione input                 | React Hook Form   |
| State Reducer            | Medium      | Logica complessa               | useState          |
| Props Getters            | Medium      | API componibile                | Render Props      |
| Compound + Context       | High        | Stato condiviso implicito      | Provider Pattern  |
| Slot Pattern             | Medium      | Layout flessibili              | Compound          |
| Polymorphic              | High        | Design system                  | Slot Pattern      |
| Headless                 | Medium      | Separazione logica/rendering   | Render Props      |
| Forwarding Refs          | Medium      | Integrazione DOM               | Callback refs     |
| Error Boundary           | Medium      | Gestione errori                | try/catch         |
| Suspense                 | Medium      | Lazy loading                   | Manual loading    |
| Optimistic Updates       | Medium      | UX reattiva                    | Server state libs |
| Derived State            | Low         | Stato derivato                 | useEffect         |
| State Machine            | High        | Workflow complessi             | Reducer Pattern   |
| Composition              | Low         | Riutilizzo flessibile          | Inheritance       |

---

## CHECKLIST COMPONENT PATTERNS

```
□ Pattern selezionato in base al use case
□ TypeScript strict mode abilitato
□ Props tipizzate correttamente
□ Context providers con error handling
□ Hooks con dependencies corrette
□ Memoization dove necessario
□ Error boundaries implementati
□ Suspense per lazy loading
□ Ref forwarding per componenti UI
□ State derivato con useMemo
```

9. ADVANCED LAYOUT PATTERNS
tsx
Copia
Scarica
// app/components/layouts/dashboard-layout.tsx
'use client';

import React, { useState, ReactNode } from 'react';
import { 
  Home, 
  BarChart3, 
  Users, 
  Settings, 
  FileText, 
  Bell, 
  Search,
  ChevronLeft,
  ChevronRight,
  User,
  HelpCircle
} from 'lucide-react';

interface DashboardWidgetProps {
  title: string;
  children: ReactNode;
  className?: string;
  colSpan?: 1 | 2;
  rowSpan?: 1 | 2;
}

interface NavItem {
  label: string;
  icon: React.ReactNode;
  href: string;
  active?: boolean;
  badge?: number;
}

const DashboardWidget: React.FC<DashboardWidgetProps> = ({ 
  title, 
  children, 
  className = '',
  colSpan = 1,
  rowSpan = 1
}) => {
  const colSpanClasses = {
    1: 'col-span-1',
    2: 'col-span-2 lg:col-span-2'
  };

  const rowSpanClasses = {
    1: 'row-span-1',
    2: 'row-span-2 lg:row-span-2'
  };

  return (
    <div className={`bg-white dark:bg-gray-800 rounded-xl border border-gray-200 dark:border-gray-700 shadow-sm overflow-hidden ${colSpanClasses[colSpan]} ${rowSpanClasses[rowSpan]} ${className}`}>
      <div className="px-6 py-4 border-b border-gray-200 dark:border-gray-700">
        <h3 className="text-lg font-semibold text-gray-900 dark:text-white">{title}</h3>
      </div>
      <div className="p-6">
        {children}
      </div>
    </div>
  );
};

interface DashboardLayoutProps {
  children?: ReactNode;
}

const DashboardLayout: React.FC<DashboardLayoutProps> = ({ children }) => {
  const [sidebarCollapsed, setSidebarCollapsed] = useState(false);
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  const navItems: NavItem[] = [
    { label: 'Dashboard', icon: <Home size={20} />, href: '#', active: true },
    { label: 'Analytics', icon: <BarChart3 size={20} />, href: '#', badge: 3 },
    { label: 'Users', icon: <Users size={20} />, href: '#' },
    { label: 'Documents', icon: <FileText size={20} />, href: '#' },
    { label: 'Settings', icon: <Settings size={20} />, href: '#' },
  ];

  const stats = [
    { label: 'Total Revenue', value: '$54,239', change: '+12.5%', positive: true },
    { label: 'Active Users', value: '2,847', change: '+5.2%', positive: true },
    { label: 'Conversion Rate', value: '3.24%', change: '-0.7%', positive: false },
    { label: 'Avg. Session', value: '4m 32s', change: '+1.2%', positive: true },
  ];

  return (
    <div className="min-h-screen bg-gray-50 dark:bg-gray-900">
      {/* Header */}
      <header className="fixed top-0 right-0 left-0 lg:left-64 z-30 bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 transition-all duration-300">
        <div className="px-4 sm:px-6 lg:px-8">
          <div className="flex items-center justify-between h-16">
            {/* Left side: Mobile menu button and breadcrumb */}
            <div className="flex items-center">
              <button
                onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
                className="lg:hidden p-2 rounded-md text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300"
              >
                <span className="sr-only">Open sidebar</span>
                <svg className="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
                </svg>
              </button>
              
              <div className="hidden sm:block ml-4">
                <nav className="flex" aria-label="Breadcrumb">
                  <ol className="flex items-center space-x-2">
                    <li>
                      <div className="flex items-center">
                        <span className="text-gray-500 dark:text-gray-400">Dashboard</span>
                      </div>
                    </li>
                    <li>
                      <div className="flex items-center">
                        <ChevronRight className="w-4 h-4 text-gray-400 mx-2" />
                        <span className="text-gray-900 dark:text-white font-medium">Overview</span>
                      </div>
                    </li>
                  </ol>
                </nav>
              </div>
            </div>

            {/* Right side: Search, notifications, profile */}
            <div className="flex items-center space-x-4">
              <div className="relative hidden md:block">
                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <Search className="h-5 w-5 text-gray-400" />
                </div>
                <input
                  type="search"
                  className="block w-64 pl-10 pr-3 py-2 border border-gray-300 dark:border-gray-600 rounded-md leading-5 bg-gray-100 dark:bg-gray-700 placeholder-gray-500 dark:placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                  placeholder="Search..."
                />
              </div>

              <button className="p-2 rounded-full text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300 relative">
                <Bell size={20} />
                <span className="absolute top-1 right-1 block h-2 w-2 rounded-full bg-red-500 ring-2 ring-white dark:ring-gray-800"></span>
              </button>

              <button className="p-2 rounded-full text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300">
                <HelpCircle size={20} />
              </button>

              <div className="relative">
                <button className="flex items-center space-x-2 focus:outline-none">
                  <div className="w-8 h-8 rounded-full bg-gradient-to-r from-blue-500 to-purple-600 flex items-center justify-center">
                    <User className="w-5 h-5 text-white" />
                  </div>
                  <span className="hidden md:block text-sm font-medium text-gray-700 dark:text-gray-300">Alex Johnson</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </header>

      {/* Sidebar */}
      <aside className={`fixed top-0 left-0 z-40 h-screen pt-16 transition-all duration-300 ${sidebarCollapsed ? 'w-20' : 'w-64'} ${mobileMenuOpen ? 'translate-x-0' : '-translate-x-full lg:translate-x-0'} bg-white dark:bg-gray-800 border-r border-gray-200 dark:border-gray-700`}>
        {/* Collapse toggle */}
        <div className="absolute -right-3 top-20 hidden lg:block">
          <button
            onClick={() => setSidebarCollapsed(!sidebarCollapsed)}
            className="p-1.5 rounded-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 shadow-md hover:shadow-lg transition-shadow"
          >
            {sidebarCollapsed ? <ChevronRight size={16} /> : <ChevronLeft size={16} />}
          </button>
        </div>

        {/* Navigation */}
        <nav className="h-full px-4 py-6 overflow-y-auto">
          <ul className="space-y-1">
            {navItems.map((item) => (
              <li key={item.label}>
                <a
                  href={item.href}
                  className={`flex items-center px-4 py-3 rounded-lg transition-colors ${item.active 
                    ? 'bg-blue-50 dark:bg-blue-900/20 text-blue-700 dark:text-blue-300' 
                    : 'text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700'}`}
                >
                  <span className="flex-shrink-0">{item.icon}</span>
                  {!sidebarCollapsed && (
                    <>
                      <span className="ml-3 flex-grow">{item.label}</span>
                      {item.badge && (
                        <span className="ml-2 px-2 py-0.5 text-xs font-medium rounded-full bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200">
                          {item.badge}
                        </span>
                      )}
                    </>
                  )}
                </a>
              </li>
            ))}
          </ul>

          {/* Secondary navigation */}
          {!sidebarCollapsed && (
            <div className="mt-12">
              <h3 className="px-4 text-xs font-semibold text-gray-500 dark:text-gray-400 uppercase tracking-wider mb-3">
                Projects
              </h3>
              <ul className="space-y-1">
                {['Website Redesign', 'Mobile App', 'Marketing Campaign'].map((project) => (
                  <li key={project}>
                    <a
                      href="#"
                      className="flex items-center px-4 py-2 text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-lg"
                    >
                      <span className="w-2 h-2 rounded-full bg-green-500 mr-3"></span>
                      {project}
                    </a>
                  </li>
                ))}
              </ul>
            </div>
          )}
        </nav>
      </aside>

      {/* Main content */}
      <main className={`pt-16 transition-all duration-300 ${sidebarCollapsed ? 'lg:pl-20' : 'lg:pl-64'}`}>
        <div className="px-4 sm:px-6 lg:px-8 py-8">
          {/* Page header */}
          <div className="mb-8">
            <h1 className="text-2xl font-bold text-gray-900 dark:text-white">Dashboard Overview</h1>
            <p className="mt-2 text-gray-600 dark:text-gray-400">
              Welcome back! Here's what's happening with your projects today.
            </p>
          </div>

          {/* Stats grid */}
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
            {stats.map((stat) => (
              <div key={stat.label} className="bg-white dark:bg-gray-800 rounded-xl border border-gray-200 dark:border-gray-700 p-6">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-sm font-medium text-gray-600 dark:text-gray-400">{stat.label}</p>
                    <p className="text-2xl font-bold text-gray-900 dark:text-white mt-2">{stat.value}</p>
                  </div>
                  <div className={`px-3 py-1 rounded-full text-sm font-medium ${stat.positive 
                    ? 'bg-green-100 text-green-800 dark:bg-green-900/20 dark:text-green-300' 
                    : 'bg-red-100 text-red-800 dark:bg-red-900/20 dark:text-red-300'}`}>
                    {stat.change}
                  </div>
                </div>
                <div className="mt-4">
                  <div className="h-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
                    <div className={`h-full ${stat.positive ? 'bg-green-500' : 'bg-red-500'}`} style={{ width: '75%' }}></div>
                  </div>
                </div>
              </div>
            ))}
          </div>

          {/* Dashboard widgets grid */}
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <DashboardWidget title="Revenue Overview" colSpan={2}>
              <div className="h-80 flex items-center justify-center bg-gray-50 dark:bg-gray-900 rounded-lg">
                <p className="text-gray-500 dark:text-gray-400">Revenue chart visualization</p>
              </div>
            </DashboardWidget>

            <DashboardWidget title="Recent Activity">
              <div className="space-y-4">
                {[
                  { user: 'Sarah Chen', action: 'added new file', time: '2 min ago' },
                  { user: 'Mike Johnson', action: 'commented on project', time: '1 hour ago' },
                  { user: 'Emma Davis', action: 'updated settings', time: '2 hours ago' },
                  { user: 'Tom Wilson', action: 'completed task', time: '5 hours ago' },
                ].map((activity) => (
                  <div key={activity.user} className="flex items-start">
                    <div className="flex-shrink-0 w-8 h-8 rounded-full bg-gradient-to-r from-blue-500 to-teal-400"></div>
                    <div className="ml-3">
                      <p className="text-sm font-medium text-gray-900 dark:text-white">
                        {activity.user} <span className="font-normal text-gray-600 dark:text-gray-400">{activity.action}</span>
                      </p>
                      <p className="text-xs text-gray-500 dark:text-gray-400 mt-1">{activity.time}</p>
                    </div>
                  </div>
                ))}
              </div>
            </DashboardWidget>

            <DashboardWidget title="Top Projects" colSpan={1} rowSpan={2}>
              <div className="space-y-4">
                {[
                  { name: 'Website Redesign', progress: 85, color: 'bg-blue-500' },
                  { name: 'Mobile App V2', progress: 60, color: 'bg-purple-500' },
                  { name: 'Marketing Campaign', progress: 45, color: 'bg-green-500' },
                  { name: 'Analytics Dashboard', progress: 30, color: 'bg-yellow-500' },
                ].map((project) => (
                  <div key={project.name}>
                    <div className="flex justify-between text-sm mb-1">
                      <span className="font-medium text-gray-900 dark:text-white">{project.name}</span>
                      <span className="text-gray-600 dark:text-gray-400">{project.progress}%</span>
                    </div>
                    <div className="h-2 bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
                      <div className={`h-full ${project.color} rounded-full`} style={{ width: `${project.progress}%` }}></div>
                    </div>
                  </div>
                ))}
              </div>
            </DashboardWidget>

            <DashboardWidget title="Team Members">
              <div className="grid grid-cols-2 gap-4">
                {['Alex Johnson', 'Sarah Chen', 'Mike Brown', 'Emma Davis', 'Tom Wilson', 'Lisa Wong'].map((member) => (
                  <div key={member} className="text-center">
                    <div className="w-12 h-12 rounded-full bg-gradient-to-r from-blue-400 to-purple-500 mx-auto mb-2"></div>
                    <p className="text-sm font-medium text-gray-900 dark:text-white">{member}</p>
                    <p className="text-xs text-gray-500 dark:text-gray-400">Developer</p>
                  </div>
                ))}
              </div>
            </DashboardWidget>

            <DashboardWidget title="Quick Actions">
              <div className="grid grid-cols-2 gap-3">
                {[
                  { icon: '📊', label: 'New Report' },
                  { icon: '👥', label: 'Add User' },
                  { icon: '📁', label: 'Upload File' },
                  { icon: '⚙️', label: 'Settings' },
                ].map((action) => (
                  <button
                    key={action.label}
                    className="flex flex-col items-center justify-center p-4 bg-gray-50 dark:bg-gray-900 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-lg transition-colors"
                  >
                    <span className="text-2xl mb-2">{action.icon}</span>
                    <span className="text-sm font-medium text-gray-900 dark:text-white">{action.label}</span>
                  </button>
                ))}
              </div>
            </DashboardWidget>
          </div>
        </div>
      </main>

      {/* Mobile menu backdrop */}
      {mobileMenuOpen && (
        <div 
          className="fixed inset-0 z-30 bg-gray-600 bg-opacity-75 lg:hidden"
          onClick={() => setMobileMenuOpen(false)}
        />
      )}
    </div>
  );
};

export default DashboardLayout;
tsx
Copia
Scarica
// app/components/layouts/split-view-layout.tsx
'use client';

import React, { useState, ReactNode } from 'react';
import { Search, Filter, ChevronRight, MoreVertical } from 'lucide-react';

interface SplitViewItem {
  id: string;
  title: string;
  subtitle: string;
  description: string;
  timestamp: string;
  unread?: boolean;
  selected?: boolean;
}

interface SplitViewLayoutProps {
  masterTitle: string;
  detailTitle: string;
  items: SplitViewItem[];
  onItemSelect?: (id: string) => void;
  detailContent?: ReactNode;
}

const SplitViewLayout: React.FC<SplitViewLayoutProps> = ({
  masterTitle,
  detailTitle,
  items,
  onItemSelect,
  detailContent
}) => {
  const [selectedId, setSelectedId] = useState<string>(items[0]?.id || '');
  const [searchQuery, setSearchQuery] = useState('');
  const [isDetailExpanded, setIsDetailExpanded] = useState(false);

  const handleItemSelect = (id: string) => {
    setSelectedId(id);
    onItemSelect?.(id);
  };

  const filteredItems = items.filter(item =>
    item.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
    item.description.toLowerCase().includes(searchQuery.toLowerCase())
  );

  const selectedItem = items.find(item => item.id === selectedId);

  return (
    <div className="flex flex-col lg:flex-row h-[600px] bg-white dark:bg-gray-900 rounded-xl border border-gray-200 dark:border-gray-700 overflow-hidden">
      {/* Master panel */}
      <div className={`lg:w-96 border-r border-gray-200 dark:border-gray-700 flex flex-col ${isDetailExpanded ? 'hidden lg:flex' : 'flex'}`}>
        {/* Header */}
        <div className="p-4 border-b border-gray-200 dark:border-gray-700">
          <div className="flex items-center justify-between mb-4">
            <h2 className="text-lg font-semibold text-gray-900 dark:text-white">{masterTitle}</h2>
            <button className="p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg">
              <Filter size={20} className="text-gray-500 dark:text-gray-400" />
            </button>
          </div>
          
          <div className="relative">
            <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-5 h-5" />
            <input
              type="text"
              placeholder="Search items..."
              className="w-full pl-10 pr-4 py-2 bg-gray-100 dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
            />
          </div>
        </div>

        {/* List */}
        <div className="flex-1 overflow-y-auto">
          {filteredItems.map((item) => (
            <div
              key={item.id}
              onClick={() => handleItemSelect(item.id)}
              className={`p-4 border-b border-gray-100 dark:border-gray-800 cursor-pointer transition-colors ${
                selectedId === item.id
                  ? 'bg-blue-50 dark:bg-blue-900/20 border-l-4 border-l-blue-500'
                  : 'hover:bg-gray-50 dark:hover:bg-gray-800'
              }`}
            >
              <div className="flex justify-between items-start mb-1">
                <div className="flex items-center">
                  {item.unread && (
                    <div className="w-2 h-2 rounded-full bg-blue-500 mr-2"></div>
                  )}
                  <h3 className="font-medium text-gray-900 dark:text-white">{item.title}</h3>
                </div>
                <span className="text-xs text-gray-500 dark:text-gray-400">{item.timestamp}</span>
              </div>
              <p className="text-sm text-gray-600 dark:text-gray-400 mb-1">{item.subtitle}</p>
              <p className="text-sm text-gray-500 dark:text-gray-400 line-clamp-2">{item.description}</p>
            </div>
          ))}
        </div>
      </div>

      {/* Detail panel */}
      <div className={`flex-1 flex flex-col ${!isDetailExpanded ? 'hidden lg:flex' : 'flex'}`}>
        {/* Detail header */}
        <div className="p-4 border-b border-gray-200 dark:border-gray-700 flex items-center justify-between">
          <div className="flex items-center">
            <button
              onClick={() => setIsDetailExpanded(false)}
              className="lg:hidden mr-3 p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg"
            >
              <ChevronRight className="w-5 h-5 text-gray-500 dark:text-gray-400 rotate-180" />
            </button>
            <div>
              <h2 className="text-lg font-semibold text-gray-900 dark:text-white">{detailTitle}</h2>
              {selectedItem && (
                <p className="text-sm text-gray-600 dark:text-gray-400">Selected: {selectedItem.title}</p>
              )}
            </div>
          </div>
          
          <div className="flex items-center space-x-2">
            <button className="p-2 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg">
              <MoreVertical size={20} className="text-gray-500 dark:text-gray-400" />
            </button>
          </div>
        </div>

        {/* Detail content */}
        <div className="flex-1 overflow-y-auto p-6">
          {detailContent || (
            <div className="max-w-3xl">
              {selectedItem ? (
                <>
                  <div className="mb-6">
                    <h1 className="text-2xl font-bold text-gray-900 dark:text-white mb-2">{selectedItem.title}</h1>
                    <div className="flex items-center text-sm text-gray-500 dark:text-gray-400 mb-4">
                      <span>{selectedItem.subtitle}</span>
                      <span className="mx-2">•</span>
                      <span>{selectedItem.timestamp}</span>
                    </div>
                    <p className="text-gray-700 dark:text-gray-300">{selectedItem.description}</p>
                  </div>
                  
                  <div className="bg-gray-50 dark:bg-gray-800 rounded-lg p-6">
                    <h3 className="font-semibold text-gray-900 dark:text-white mb-4">Details</h3>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <p className="text-sm text-gray-600 dark:text-gray-400">Status</p>
                        <p className="font-medium text-green-600 dark:text-green-400">Active</p>
                      </div>
                      <div>
                        <p className="text-sm text-gray-600 dark:text-gray-400">Priority</p>
                        <p className="font-medium text-yellow-600 dark:text-yellow-400">High</p>
                      </div>
                      <div>
                        <p className="text-sm text-gray-600 dark:text-gray-400">Assigned to</p>
                        <p className="font-medium text-gray-900 dark:text-white">Alex Johnson</p>
                      </div>
                      <div>
                        <p className="text-sm text-gray-600 dark:text-gray-400">Due Date</p>
                        <p className="font-medium text-gray-900 dark:text-white">Dec 15, 2024</p>
                      </div>
                    </div>
                  </div>
                </>
              ) : (
                <div className="text-center py-12">
                  <div className="text-gray-400 dark:text-gray-500 mb-4">Select an item to view details</div>
                </div>
              )}
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export { SplitViewLayout };
export type { SplitViewItem };
tsx
Copia
Scarica
// app/components/layouts/sticky-header-layout.tsx
'use client';

import React, { useState, useEffect, ReactNode } from 'react';
import { Menu, X, ChevronDown, ChevronUp } from 'lucide-react';

interface StickyHeaderLayoutProps {
  children: ReactNode;
  headerContent?: ReactNode;
  showScrollProgress?: boolean;
}

const StickyHeaderLayout: React.FC<StickyHeaderLayoutProps> = ({
  children,
  headerContent,
  showScrollProgress = true
}) => {
  const [isScrolled, setIsScrolled] = useState(false);
  const [scrollProgress, setScrollProgress] = useState(0);
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  useEffect(() => {
    const handleScroll = () => {
      const scrolled = window.scrollY > 10;
      setIsScrolled(scrolled);

      if (showScrollProgress) {
        const winScroll = document.body.scrollTop || document.documentElement.scrollTop;
        const height = document.documentElement.scrollHeight - document.documentElement.clientHeight;
        const scrolledPercent = (winScroll / height) * 100;
        setScrollProgress(scrolledPercent);
      }
    };

    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [showScrollProgress]);

  const navItems = [
    { label: 'Home', href: '#' },
    { label: 'Products', href: '#', children: true },
    { label: 'Solutions', href: '#', children: true },
    { label: 'Pricing', href: '#' },
    { label: 'Documentation', href: '#' },
    { label: 'Company', href: '#', children: true },
  ];

  return (
    <div className="min-h-screen">
      {/* Sticky header */}
      <header
        className={`sticky top-0 z-50 transition-all duration-300 ${
          isScrolled
            ? 'bg-white/95 dark:bg-gray-900/95 backdrop-blur-md border-b border-gray-200 dark:border-gray-800 shadow-lg'
            : 'bg-white dark:bg-gray-900 border-b border-gray-200 dark:border-gray-800'
        }`}
      >
        {/* Scroll progress bar */}
        {showScrollProgress && (
          <div className="h-1 w-full bg-gray-100 dark:bg-gray-800">
            <div
              className="h-full bg-gradient-to-r from-blue-500 to-purple-600 transition-all duration-300"
              style={{ width: `${scrollProgress}%` }}
            />
          </div>
        )}

        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex items-center justify-between h-16">
            {/* Logo */}
            <div className="flex items-center">
              <div className="text-2xl font-bold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
                Acme Inc.
              </div>
            </div>

            {/* Desktop navigation */}
            <nav className="hidden md:flex items-center space-x-1">
              {navItems.map((item) => (
                <div key={item.label} className="relative group">
                  <a
                    href={item.href}
                    className="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800 flex items-center"
                  >
                    {item.label}
                    {item.children && (
                      <ChevronDown className="ml-1 w-4 h-4 group-hover:rotate-180 transition-transform" />
                    )}
                  </a>
                  {item.children && (
                    <div className="absolute left-0 mt-2 w-56 opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all duration-200">
                      <div className="py-2 bg-white dark:bg-gray-800 rounded-lg shadow-xl border border-gray-200 dark:border-gray-700">
                        {[1, 2, 3].map((i) => (
                          <a
                            key={i}
                            href="#"
                            className="block px-4 py-2 text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700"
                          >
                            Subitem {i}
                          </a>
                        ))}
                      </div>
                    </div>
                  )}
                </div>
              ))}
            </nav>

            {/* CTA buttons */}
            <div className="hidden md:flex items-center space-x-4">
              <button className="px-4 py-2 text-sm font-medium text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400">
                Sign in
              </button>
              <button className="px-4 py-2 bg-gradient-to-r from-blue-600 to-purple-600 text-white rounded-lg hover:opacity-90 transition-opacity">
                Get Started
              </button>
            </div>

            {/* Mobile menu button */}
            <button
              className="md:hidden p-2 rounded-lg text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300"
              onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
            >
              {mobileMenuOpen ? <X size={24} /> : <Menu size={24} />}
            </button>
          </div>
        </div>

        {/* Mobile menu */}
        {mobileMenuOpen && (
          <div className="md:hidden border-t border-gray-200 dark:border-gray-800 bg-white dark:bg-gray-900">
            <div className="px-4 py-3 space-y-1">
              {navItems.map((item) => (
                <a
                  key={item.label}
                  href={item.href}
                  className="block px-4 py-3 text-base font-medium text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg"
                >
                  {item.label}
                </a>
              ))}
              <div className="pt-4 border-t border-gray-200 dark:border-gray-800 space-y-3">
                <button className="w-full px-4 py-2 text-base font-medium text-gray-700 dark:text-gray-300 hover:text-blue-600 dark:hover:text-blue-400">
                  Sign in
                </button>
                <button className="w-full px-4 py-2 bg-gradient-to-r from-blue-600 to-purple-600 text-white rounded-lg hover:opacity-90 transition-opacity">
                  Get Started
                </button>
              </div>
            </div>
          </div>
        )}
      </header>

      {/* Custom header content or default */}
      {headerContent ? (
        headerContent
      ) : (
        <div className="relative overflow-hidden bg-gradient-to-br from-blue-50 to-purple-50 dark:from-gray-900 dark:to-gray-800">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-24">
            <div className="text-center">
              <h1 className="text-4xl md:text-6xl font-bold text-gray-900 dark:text-white mb-6">
                Build amazing products
                <span className="block bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
                  faster than ever
                </span>
              </h1>
              <p className="text-xl text-gray-600 dark:text-gray-400 max-w-3xl mx-auto mb-10">
                A complete design system and component library for building modern web applications with React and Next.js.
              </p>
              <div className="flex flex-col sm:flex-row gap-4 justify-center">
                <button className="px-8 py-3 bg-gradient-to-r from-blue-600 to-purple-600 text-white rounded-lg hover:opacity-90 transition-opacity text-lg font-medium">
                  Get Started Free
                </button>
                <button className="px-8 py-3 border-2 border-gray-300 dark:border-gray-700 text-gray-700 dark:text-gray-300 rounded-lg hover:border-blue-500 hover:text-blue-600 dark:hover:text-blue-400 transition-colors text-lg font-medium">
                  View Demo
                </button>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* Main content */}
      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
        {children}
      </main>

      {/* Scroll to top button */}
      {isScrolled && (
        <button
          onClick={() => window.scrollTo({ top: 0, behavior: 'smooth' })}
          className="fixed bottom-8 right-8 p-3 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-700 rounded-full shadow-lg hover:shadow-xl transition-shadow z-40"
        >
          <ChevronUp className="w-5 h-5 text-gray-700 dark:text-gray-300" />
        </button>
      )}
    </div>
  );
};

export { StickyHeaderLayout };
10. DATA DISPLAY PATTERNS
tsx
Copia
Scarica
// app/components/data-display/data-table.tsx
'use client';

import React, { useState, useMemo, ReactNode } from 'react';
import { 
  ChevronUp, 
  ChevronDown, 
  ChevronsUpDown, 
  Search, 
  Filter, 
  MoreVertical,
  Download,
  Eye,
  Edit,
  Trash2,
  ArrowLeft,
  ArrowRight
} from 'lucide-react';

type SortDirection = 'asc' | 'desc' | null;

interface Column<T> {
  key: keyof T | string;
  header: string;
  sortable?: boolean;
  filterable?: boolean;
  width?: string;
  render?: (value: any, row: T) => ReactNode;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  onSort?: (key: keyof T, direction: SortDirection) => void;
  onAction?: (action: string, row: T) => void;
  loading?: boolean;
}

const DataTable = <T extends { id: string | number }>({
  data,
  columns,
  pageSize = 10,
  selectable = false,
  onRowClick,
  onSelectionChange,
  onSort,
  onAction,
  loading = false
}: DataTableProps<T>) => {
  const [currentPage, setCurrentPage] = useState(1);
  const [selectedRows, setSelectedRows] = useState<Set<string | number>>(new Set());
  const [sortConfig, setSortConfig] = useState<{ key: keyof T; direction: SortDirection } | null>(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [columnFilters, setColumnFilters] = useState<Record<string, string>>({});

  // Pagination calculations
  const totalPages = Math.ceil(data.length / pageSize);
  const startIndex = (currentPage - 1) * pageSize;
  const endIndex = startIndex + pageSize;

  // Filtering and sorting
  const processedData = useMemo(() => {
    let result = [...data];

    //

border-gray-200 dark:border-gray-700 p-6">
      <div className="flex items-start justify-between mb-4">
        <div

## PATTERN 3: Higher-Order Components (HOC)

**Metadata:**

| Campo         | Valore                                        |
| ------------- | --------------------------------------------- |
| Categoria     | Structural                                    |
| Complessità   | Medium                                        |
| React Version | 18+                                           |
| Use Case      | Condivisione logica tra componenti non correlati |

**Implementazione:**

```typescript
import React, { useState, useEffect } from 'react';

interface WithLoadingIndicatorProps {
  isLoading: boolean;
  children: React.ReactNode;
}

const withLoadingIndicator = <P extends {}>(WrappedComponent: React.ComponentType<P>) => {
  const EnhancedComponent = ({ isLoading, ...props }: WithLoadingIndicatorProps & P) => {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <WrappedComponent {...props} />;
  };
  return EnhancedComponent;
};

export const MyComponent = withLoadingIndicator(({ name }: { name: string }) => {
  return <div>Hello, {name}!</div>;
});
```

**Quando Usare:** ✅ Condivisione logica tra componenti non correlati | ✅ Gestione stato di caricamento

**Quando Evitare:** ❌ Componenti semplici | ❌ Performance critiche

## PATTERN 4: React Hooks

**Metadata:**

| Campo         | Valore                                        |
| ------------- | --------------------------------------------- |
| Categoria     | Behavioral                                    |
| Complessità   | Medium                                        |
| React Version | 18+                                           |
| Use Case      | Gestione stato e effetti collaterali in componenti funzionali |

**Implementazione:**

```typescript
import { useState, useEffect } from 'react';

export const useFetchData = (url: string) => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setIsLoading(true);
      try {
        const response = await fetch(url);
        const data = await response.json();
        setData(data);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    };
    fetchData();
  }, [url]);

  return { data, error, isLoading };
};
```

**Quando Usare:** ✅ Gestione stato e effetti collaterali in componenti funzionali | ✅ Fetching dati da API

**Quando Evitare:** ❌ Componenti classici | ❌ Performance critiche

## Testing Patterns con Vitest

### Test di un componente React

```typescript
import { render, fireEvent, waitFor } from '@testing-library/react';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  it('renders correctly', () => {
    const { getByText } = render(<MyComponent />);
    expect(getByText('Hello, World!')).toBeInTheDocument();
  });

  it('handles click event', () => {
    const { getByText } = render(<MyComponent />);
    const button = getByText('Click me!');
    fireEvent.click(button);
    expect(getByText('Button clicked!')).toBeInTheDocument();
  });
});
```

### Test di un hook React

```typescript
import { renderHook, act } from '@testing-library/react-hooks';
import { useFetchData } from './useFetchData';

describe('useFetchData', () => {
  it('fetches data correctly', async () => {
    const { result, waitForNextUpdate } = renderHook(() => useFetchData('https://api.example.com/data'));
    await waitForNextUpdate();
    expect(result.current.data).toEqual({ /* expected data */ });
  });

  it('handles error correctly', async () => {
    const { result, waitForNextUpdate } = renderHook(() => useFetchData('https://api.example.com/error'));
    await waitForNextUpdate();
    expect(result.current.error).toBeInstanceOf(Error);
  });
});
```

## Best Practices

### ✅ DO

* Utilizzare componenti funzionali invece di classici
* Utilizzare hooks per gestire stato e effetti collaterali
* Utilizzare librerie di testing come Vitest per testare i componenti

### ❌ DON'T

* Utilizzare componenti classici per nuove implementazioni
* Utilizzare stato locale in componenti funzionali senza hooks
* Non testare i componenti prima di deployarli

## Common Pitfalls & Troubleshooting

* **Stato non aggiornato**: Assicurarsi di utilizzare `useState` e `useEffect` correttamente per gestire lo stato dei componenti.
* **Errori di rendering**: Verificare che i componenti siano correttamente importati e utilizzati nel codice.
* **Problemi di performance**: Utilizzare strumenti di profiling per identificare le cause di problemi di performance e ottimizzare il codice di conseguenza.

## Migration/Upgrade Patterns

* **Migrazione da React 17 a React 18**: Utilizzare la documentazione ufficiale di React per eseguire la migrazione e risolvere eventuali problemi.
* **Aggiornamento di librerie**: Utilizzare npm o yarn per aggiornare le librerie e risolvere eventuali problemi di compatibilità.
* **Rifattorizzazione del codice**: Utilizzare strumenti di analisi del codice per identificare aree di miglioramento e rifattorizzare il codice per renderlo più efficiente e facile da mantenere.