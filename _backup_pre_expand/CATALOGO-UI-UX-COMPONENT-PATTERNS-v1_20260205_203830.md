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
