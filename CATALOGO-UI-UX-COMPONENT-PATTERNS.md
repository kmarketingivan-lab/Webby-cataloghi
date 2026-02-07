# CATALOGO-UI-UX-COMPONENT-PATTERNS

CATALOGO-UI-UX-COMPONENT-PATTERNS per Next.js 14 + shadcn
§ COMPOUND COMPONENTS
Pattern explanation

I componenti compound permettono di creare componenti che gestiscono stato e logica complessa, mentre espongono API dichiarative e flessibili. Questo pattern è particolarmente utile per componenti UI complessi come tabelle, form, o componenti di navigazione.

typescript
// Esempio: Tabs con Compound Components
import React, { createContext, useContext, useState } from 'react';

interface TabsContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = createContext<TabsContextType | undefined>(undefined);

interface TabsProps {
  defaultValue: string;
  children: React.ReactNode;
}

export function Tabs({ defaultValue, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="w-full">{children}</div>
    </TabsContext.Provider>
  );
}

export function TabsList({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex space-x-2 border-b border-gray-200">{children}</div>
  );
}

export function TabsTrigger({ value, children }: { 
  value: string; 
  children: React.ReactNode;
}) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabsTrigger must be used within Tabs');

  const isActive = context.activeTab === value;

  return (
    <button
      className={`px-4 py-2 font-medium text-sm transition-colors ${
        isActive
          ? 'text-primary border-b-2 border-primary'
          : 'text-gray-600 hover:text-gray-900'
      }`}
      onClick={() => context.setActiveTab(value)}
    >
      {children}
    </button>
  );
}

export function TabsContent({ value, children }: { 
  value: string; 
  children: React.ReactNode;
}) {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabsContent must be used within Tabs');

  if (context.activeTab !== value) return null;

  return <div className="p-4">{children}</div>;
}

// Utilizzo
function ExampleTabs() {
  return (
    <Tabs defaultValue="profile">
      <TabsList>
        <TabsTrigger value="profile">Profile</TabsTrigger>
        <TabsTrigger value="settings">Settings</TabsTrigger>
        <TabsTrigger value="billing">Billing</TabsTrigger>
      </TabsList>
      <TabsContent value="profile">Profile content</TabsContent>
      <TabsContent value="settings">Settings content</TabsContent>
      <TabsContent value="billing">Billing content</TabsContent>
    </Tabs>
  );
}
Context-based sharing

Il pattern utilizza React Context per condividere lo stato tra i componenti figli senza passare props manualmente a ogni livello.

typescript
// Pattern avanzato: Accordion con Context
import React, { createContext, useContext, useState } from 'react';

interface AccordionContextType {
  openItems: Set<string>;
  toggleItem: (id: string) => void;
  allowMultiple?: boolean;
}

const AccordionContext = createContext<AccordionContextType | undefined>(undefined);

export function Accordion({ 
  children, 
  allowMultiple = false 
}: { 
  children: React.ReactNode;
  allowMultiple?: boolean;
}) {
  const [openItems, setOpenItems] = useState<Set<string>>(new Set());

  const toggleItem = (id: string) => {
    setOpenItems(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        if (!allowMultiple) next.clear();
        next.add(id);
      }
      return next;
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggleItem, allowMultiple }}>
      <div className="space-y-2">{children}</div>
    </AccordionContext.Provider>
  );
}
Flexible composition

I componenti compound offrono massima flessibilità nella composizione, permettendo di riordinare e personalizzare i componenti figli.

typescript
// Componente Select con massima flessibilità
import React, { createContext, useContext, useState, useRef, useEffect } from 'react';

interface SelectContextType {
  isOpen: boolean;
  selectedValue: string | null;
  open: () => void;
  close: () => void;
  selectValue: (value: string) => void;
}

const SelectContext = createContext<SelectContextType | undefined>(undefined);

export function Select({ children, defaultValue = null }: {
  children: React.ReactNode;
  defaultValue?: string | null;
}) {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedValue, setSelectedValue] = useState<string | null>(defaultValue);

  return (
    <SelectContext.Provider
      value={{
        isOpen,
        selectedValue,
        open: () => setIsOpen(true),
        close: () => setIsOpen(false),
        selectValue: (value) => {
          setSelectedValue(value);
          setIsOpen(false);
        },
      }}
    >
      <div className="relative">{children}</div>
    </SelectContext.Provider>
  );
}
§ RENDER PROPS
When to use

Le render props sono utili quando:

Devi condividere logica tra componenti con UI diverse

Hai bisogno di massima flessibilità nel rendering

Vuoi evitare prop drilling complesso

L'ereditarietà non è adatta (preferisci composizione)

typescript
// Pattern base: Render Props per dati asincroni
import React, { useState, useEffect } from 'react';

interface FetchProps<T> {
  url: string;
  children: (state: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

export function Fetch<T>({ url, children }: FetchProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await fetch(url);
      const result = await response.json();
      setData(result);
      setError(null);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return (
    <>
      {children({
        data,
        loading,
        error,
        refetch: fetchData
      })}
    </>
  );
}

// Utilizzo
function UserProfile({ userId }: { userId: string }) {
  return (
    <Fetch url={`/api/users/${userId}`}>
      {({ data, loading, error, refetch }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        return (
          <div>
            <h1>{data?.name}</h1>
            <button onClick={refetch}>Refresh</button>
          </div>
        );
      }}
    </Fetch>
  );
}
Implementation pattern

Implementazione type-safe delle render props con supporto per props multiple.

typescript
// Pattern avanzato: List con render props
import React from 'react';

interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  renderEmpty?: () => React.ReactNode;
  renderHeader?: () => React.ReactNode;
  renderFooter?: () => React.ReactNode;
  className?: string;
}

export function List<T>({
  items,
  renderItem,
  renderEmpty,
  renderHeader,
  renderFooter,
  className = '',
}: ListProps<T>) {
  return (
    <div className={`space-y-2 ${className}`}>
      {renderHeader?.()}
      {items.length === 0 ? (
        renderEmpty ? renderEmpty() : <div>No items found</div>
      ) : (
        <ul className="space-y-2">
          {items.map((item, index) => (
            <li key={index}>{renderItem(item, index)}</li>
          ))}
        </ul>
      )}
      {renderFooter?.()}
    </div>
  );
}

// Utilizzo con tipi complessi
interface Product {
  id: string;
  name: string;
  price: number;
}

function ProductList({ products }: { products: Product[] }) {
  return (
    <List
      items={products}
      renderHeader={() => <h2 className="text-xl font-bold">Products</h2>}
      renderEmpty={() => <div>No products available</div>}
      renderItem={(product) => (
        <div className="flex justify-between p-4 border rounded">
          <span>{product.name}</span>
          <span>${product.price.toFixed(2)}</span>
        </div>
      )}
      renderFooter={() => (
        <div className="text-sm text-gray-500">
          Showing {products.length} products
        </div>
      )}
    />
  );
}
Type-safe render props

Implementazione con TypeScript generics per massima type safety.

typescript
// Table component con render props type-safe
import React from 'react';

interface Column<T> {
  key: keyof T;
  header: string;
  render?: (value: T[keyof T], item: T) => React.ReactNode;
}

interface TableProps<T> {
  data: T[];
  columns: Column<T>[];
  renderRowActions?: (item: T) => React.ReactNode;
  renderEmptyState?: () => React.ReactNode;
  onRowClick?: (item: T) => void;
}

export function Table<T extends Record<string, any>>({
  data,
  columns,
  renderRowActions,
  renderEmptyState,
  onRowClick,
}: TableProps<T>) {
  if (data.length === 0) {
    return renderEmptyState ? renderEmptyState() : <div>No data</div>;
  }

  return (
    <div className="overflow-x-auto">
      <table className="min-w-full divide-y divide-gray-200">
        <thead>
          <tr>
            {columns.map((column) => (
              <th
                key={column.key as string}
                className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
              >
                {column.header}
              </th>
            ))}
            {renderRowActions && <th className="px-6 py-3">Actions</th>}
          </tr>
        </thead>
        <tbody className="bg-white divide-y divide-gray-200">
          {data.map((item, index) => (
            <tr
              key={index}
              onClick={() => onRowClick?.(item)}
              className={onRowClick ? 'cursor-pointer hover:bg-gray-50' : ''}
            >
              {columns.map((column) => (
                <td key={column.key as string} className="px-6 py-4 whitespace-nowrap">
                  {column.render
                    ? column.render(item[column.key], item)
                    : String(item[column.key])}
                </td>
              ))}
              {renderRowActions && (
                <td className="px-6 py-4 whitespace-nowrap">
                  {renderRowActions(item)}
                </td>
              )}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
§ HIGHER-ORDER COMPONENTS
HOC pattern in React

Gli Higher-Order Components sono funzioni che prendono un componente e restituiscono un nuovo componente con funzionalità aggiuntive.

typescript
// HOC base: withAuthentication
import React, { useEffect, useState, ComponentType } from 'react';
import { useRouter } from 'next/router';

interface WithAuthProps {
  isAuthenticated: boolean;
}

export function withAuth<P extends object>(
  WrappedComponent: ComponentType<P & WithAuthProps>
) {
  const ComponentWithAuth = (props: P) => {
    const router = useRouter();
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      const checkAuth = async () => {
        const token = localStorage.getItem('auth_token');
        if (!token) {
          router.push('/login');
        } else {
          setIsAuthenticated(true);
        }
        setLoading(false);
      };

      checkAuth();
    }, [router]);

    if (loading) {
      return <div>Loading...</div>;
    }

    if (!isAuthenticated) {
      return null;
    }

    return <WrappedComponent {...props} isAuthenticated={isAuthenticated} />;
  };

  ComponentWithAuth.displayName = `withAuth(${
    WrappedComponent.displayName || WrappedComponent.name || 'Component'
  })`;

  return ComponentWithAuth;
}

// Utilizzo
interface DashboardProps extends WithAuthProps {
  title: string;
}

function Dashboard({ title, isAuthenticated }: DashboardProps) {
  return (
    <div>
      <h1>{title}</h1>
      <p>Authenticated: {isAuthenticated ? 'Yes' : 'No'}</p>
    </div>
  );
}

export const ProtectedDashboard = withAuth(Dashboard);
Type safety

Implementazione type-safe per HOC con props preservate.

typescript
// HOC type-safe con props forwarding
import React, { ComponentType, useEffect } from 'react';

// Tipo utility per rimuovere props dal wrapped component
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// HOC per logging
export function withLogging<P extends { componentName?: string }>(
  WrappedComponent: ComponentType<P>,
  componentName?: string
) {
  const ComponentWithLogging = (props: P) => {
    useEffect(() => {
      console.log(`[${componentName || 'Component'}] Mounted`);
      
      return () => {
        console.log(`[${componentName || 'Component'}] Unmounted`);
      };
    }, []);

    useEffect(() => {
      console.log(`[${componentName || 'Component'}] Props updated:`, props);
    }, [props]);

    return <WrappedComponent {...props} />;
  };

  ComponentWithLogging.displayName = `withLogging(${
    componentName || WrappedComponent.displayName || WrappedComponent.name || 'Component'
  })`;

  return ComponentWithLogging;
}

// HOC per loading state
export function withLoading<P extends { isLoading?: boolean }>(
  WrappedComponent: ComponentType<Omit<P, 'isLoading'>>
) {
  const ComponentWithLoading = (props: P) => {
    if (props.isLoading) {
      return (
        <div className="flex items-center justify-center p-8">
          <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary"></div>
        </div>
      );
    }

    // Rimuovi isLoading dalle props passate al componente wrappato
    const { isLoading, ...restProps } = props;
    return <WrappedComponent {...(restProps as Omit<P, 'isLoading'>)} />;
  };

  return ComponentWithLoading;
}
Composition vs HOC

Quando preferire la composizione rispetto agli HOC.

typescript
// Approccio con HOC (più verboso)
import React from 'react';

// Multiple HOC example
function withTheme<P extends object>(WrappedComponent: ComponentType<P>) {
  return (props: P) => {
    const theme = 'dark'; // In realtà verrebbe da un context
    return <WrappedComponent {...props} theme={theme} />;
  };
}

function withUser<P extends object>(WrappedComponent: ComponentType<P>) {
  return (props: P) => {
    const user = { name: 'John', id: '123' }; // In realtà da auth context
    return <WrappedComponent {...props} user={user} />;
  };
}

// Approccio con Hook + Composizione (preferito)
import { useTheme, useUser } from '@/hooks';

function ProfileComponent() {
  const theme = useTheme();
  const user = useUser();
  
  // Usa theme e user direttamente
  return <div>Profile for {user.name}</div>;
}

// Pattern moderno: Custom Hook + Component
export function useProfileData() {
  const theme = useTheme();
  const user = useUser();
  const [profile, setProfile] = useState(null);

  useEffect(() => {
    // Fetch profile data
  }, [user.id]);

  return { theme, user, profile };
}

function Profile() {
  const { theme, user, profile } = useProfileData();
  // Logica componente
}
When to use

Usa HOC quando:

Devi aggiungere la stessa funzionalità a molti componenti

La logica è complessa e può essere isolata

Stai usando librerie che seguono questo pattern

Non puoi usare Hook per limitazioni (es: class components)

Evita HOC quando:

Puoi usare Hook custom

La logica è specifica per un componente

Vuoi evitare il nesting di componenti

Hai bisogno di debugging più semplice

§ HEADLESS COMPONENTS
Logic-only components

Componenti che forniscono solo la logica, lasciando lo styling completamente all'utente.

typescript
// Headless Toggle component
import React, { createContext, useContext, useState, useCallback } from 'react';

interface ToggleContextType {
  on: boolean;
  toggle: () => void;
  setOn: (value: boolean) => void;
}

const ToggleContext = createContext<ToggleContextType | undefined>(undefined);

export function useToggle() {
  const context = useContext(ToggleContext);
  if (!context) {
    throw new Error('useToggle must be used within a ToggleProvider');
  }
  return context;
}

export function ToggleProvider({
  initialOn = false,
  children,
}: {
  initialOn?: boolean;
  children: React.ReactNode;
}) {
  const [on, setOn] = useState(initialOn);

  const toggle = useCallback(() => setOn(prev => !prev), []);

  const value = {
    on,
    toggle,
    setOn,
  };

  return (
    <ToggleContext.Provider value={value}>
      {children}
    </ToggleContext.Provider>
  );
}

// Componenti headless per l'uso
export function ToggleButton({
  children,
  ...props
}: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  const { on, toggle } = useToggle();
  
  return (
    <button
      onClick={toggle}
      data-state={on ? 'on' : 'off'}
      aria-pressed={on}
      {...props}
    >
      {children}
    </button>
  );
}

export function ToggleOn({ children }: { children: React.ReactNode }) {
  const { on } = useToggle();
  return on ? <>{children}</> : null;
}

export function ToggleOff({ children }: { children: React.ReactNode }) {
  const { on } = useToggle();
  return !on ? <>{children}</> : null;
}

// Utilizzo personalizzato
function CustomToggle() {
  return (
    <ToggleProvider>
      <div className="flex items-center space-x-4">
        <ToggleButton className="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300">
          Toggle
        </ToggleButton>
        <ToggleOn>
          <span className="text-green-600">✓ On</span>
        </ToggleOn>
        <ToggleOff>
          <span className="text-red-600">✗ Off</span>
        </ToggleOff>
      </div>
    </ToggleProvider>
  );
}
Custom styling

Pattern per headless components che supportano stili completamente personalizzabili.

typescript
// Headless Modal Component
import React, { createContext, useContext, useState, useCallback, useEffect } from 'react';

interface ModalContextType {
  isOpen: boolean;
  open: () => void;
  close: () => void;
  toggle: () => void;
}

const ModalContext = createContext<ModalContextType | undefined>(undefined);

export function useModal() {
  const context = useContext(ModalContext);
  if (!context) {
    throw new Error('useModal must be used within a ModalProvider');
  }
  return context;
}

export function ModalProvider({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);

  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(prev => !prev), []);

  // Close on escape key
  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') close();
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      return () => document.removeEventListener('keydown', handleEscape);
    }
  }, [isOpen, close]);

  const value = {
    isOpen,
    open,
    close,
    toggle,
  };

  return (
    <ModalContext.Provider value={value}>
      {children}
    </ModalContext.Provider>
  );
}

// Headless components per costruire qualsiasi tipo di modal
export function ModalTrigger({ children, ...props }: any) {
  const { open } = useModal();
  return React.cloneElement(children, {
    onClick: (e: React.MouseEvent) => {
      children.props.onClick?.(e);
      open();
    },
    ...props,
  });
}

export function ModalContent({ children }: { children: React.ReactNode }) {
  const { isOpen, close } = useModal();

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      {/* Backdrop */}
      <div 
        className="absolute inset-0 bg-black/50" 
        onClick={close}
      />
      
      {/* Content container */}
      <div className="relative bg-white rounded-lg shadow-xl">
        {children}
      </div>
    </div>
  );
}

export function ModalClose({ children }: { children: React.ReactNode }) {
  const { close } = useModal();
  return React.cloneElement(children as React.ReactElement, {
    onClick: (e: React.MouseEvent) => {
      (children as React.ReactElement).props.onClick?.(e);
      close();
    },
  });
}
Radix UI patterns

Pattern ispirati a Radix UI per componenti accessibili e headless.

typescript
// Radix-inspired Dialog Primitives
import React, { createContext, useContext, useRef } from 'react';

interface DialogContextType {
  isOpen: boolean;
  openDialog: () => void;
  closeDialog: () => void;
  triggerRef: React.RefObject<HTMLButtonElement>;
}

const DialogContext = createContext<DialogContextType | undefined>(undefined);

export function Dialog({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = React.useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);

  const openDialog = () => setIsOpen(true);
  const closeDialog = () => {
    setIsOpen(false);
    // Restore focus to trigger
    setTimeout(() => triggerRef.current?.focus(), 0);
  };

  // Close on escape
  React.useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') closeDialog();
    };

    if (isOpen) {
      document.addEventListener('keydown', handleKeyDown);
      return () => document.removeEventListener('keydown', handleKeyDown);
    }
  }, [isOpen]);

  // Trap focus inside dialog
  React.useEffect(() => {
    if (isOpen) {
      const focusableElements = document.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0] as HTMLElement;
      const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

      const handleTabKey = (e: KeyboardEvent) => {
        if (e.key === 'Tab') {
          if (e.shiftKey && document.activeElement === firstElement) {
            e.preventDefault();
            lastElement.focus();
          } else if (!e.shiftKey && document.activeElement === lastElement) {
            e.preventDefault();
            firstElement.focus();
          }
        }
      };

      document.addEventListener('keydown', handleTabKey);
      return () => document.removeEventListener('keydown', handleTabKey);
    }
  }, [isOpen]);

  return (
    <DialogContext.Provider value={{ isOpen, openDialog, closeDialog, triggerRef }}>
      {children}
    </DialogContext.Provider>
  );
}

export function DialogTrigger({ children }: { children: React.ReactNode }) {
  const context = useContext(DialogContext);
  if (!context) throw new Error('DialogTrigger must be used within Dialog');

  return React.cloneElement(children as React.ReactElement, {
    ref: context.triggerRef,
    onClick: () => context.openDialog(),
  });
}

export function DialogContent({ children }: { children: React.ReactNode }) {
  const context = useContext(DialogContext);
  if (!context) throw new Error('DialogContent must be used within Dialog');
  if (!context.isOpen) return null;

  return (
    <div 
      role="dialog"
      aria-modal="true"
      className="fixed inset-0 z-50 flex items-center justify-center p-4"
    >
      <div 
        className="absolute inset-0 bg-black/50"
        onClick={context.closeDialog}
        aria-hidden="true"
      />
      <div 
        className="relative bg-white rounded-lg shadow-xl"
        role="document"
      >
        {children}
      </div>
    </div>
  );
}
Downshift patterns

Pattern ispirati a Downshift per componenti di selezione headless.

typescript
// Downshift-inspired Combobox
import React, { useState, useRef, useEffect, KeyboardEvent } from 'react';

interface UseComboboxProps<T> {
  items: T[];
  itemToString: (item: T | null) => string;
  initialSelectedItem?: T | null;
  onSelectedItemChange?: (item: T | null) => void;
}

export function useCombobox<T>({
  items,
  itemToString,
  initialSelectedItem = null,
  onSelectedItemChange,
}: UseComboboxProps<T>) {
  const [isOpen, setIsOpen] = useState(false);
  const [inputValue, setInputValue] = useState('');
  const [selectedItem, setSelectedItem] = useState<T | null>(initialSelectedItem);
  const [highlightedIndex, setHighlightedIndex] = useState(-1);
  const inputRef = useRef<HTMLInputElement>(null);

  const filteredItems = items.filter(item =>
    itemToString(item).toLowerCase().includes(inputValue.toLowerCase())
  );

  const openMenu = () => setIsOpen(true);
  const closeMenu = () => setIsOpen(false);

  const selectItem = (item: T) => {
    setSelectedItem(item);
    setInputValue(itemToString(item));
    onSelectedItemChange?.(item);
    closeMenu();
    inputRef.current?.focus();
  };

  const handleInputChange = (value: string) => {
    setInputValue(value);
    setHighlightedIndex(-1);
    if (!isOpen) openMenu();
  };

  const handleKeyDown = (e: KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setHighlightedIndex(prev =>
          prev < filteredItems.length - 1 ? prev + 1 : prev
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setHighlightedIndex(prev => (prev > 0 ? prev - 1 : prev));
        break;
      case 'Enter':
        e.preventDefault();
        if (highlightedIndex >= 0 && filteredItems[highlightedIndex]) {
          selectItem(filteredItems[highlightedIndex]);
        }
        break;
      case 'Escape':
        e.preventDefault();
        closeMenu();
        break;
    }
  };

  useEffect(() => {
    if (isOpen && filteredItems.length > 0 && highlightedIndex === -1) {
      setHighlightedIndex(0);
    }
  }, [isOpen, filteredItems.length]);

  return {
    // State
    isOpen,
    inputValue,
    selectedItem,
    highlightedIndex,
    filteredItems,
    
    // Refs
    inputRef,
    
    // Actions
    openMenu,
    closeMenu,
    selectItem,
    handleInputChange,
    handleKeyDown,
    
    // Props getters
    getInputProps: () => ({
      ref: inputRef,
      value: inputValue,
      onChange: (e: React.ChangeEvent<HTMLInputElement>) =>
        handleInputChange(e.target.value),
      onFocus: openMenu,
      onKeyDown: handleKeyDown,
      'aria-autocomplete': 'list',
      'aria-expanded': isOpen,
      'aria-controls': 'combobox-menu',
    }),
    
    getMenuProps: () => ({
      id: 'combobox-menu',
      role: 'listbox',
    }),
    
    getItemProps: (item: T, index: number) => ({
      key: index,
      role: 'option',
      'aria-selected': index === highlightedIndex,
      onClick: () => selectItem(item),
      className: index === highlightedIndex ? 'bg-blue-100' : '',
    }),
  };
}

// Utilizzo
function CountryCombobox() {
  const countries = [
    { code: 'US', name: 'United States' },
    { code: 'CA', name: 'Canada' },
    { code: 'MX', name: 'Mexico' },
    { code: 'GB', name: 'United Kingdom' },
  ];

  const combobox = useCombobox({
    items: countries,
    itemToString: (item) => item?.name || '',
  });

  return (
    <div className="relative">
      <input
        {...combobox.getInputProps()}
        className="w-full p-2 border rounded"
        placeholder="Select a country"
      />
      {combobox.isOpen && (
        <ul
          {...combobox.getMenuProps()}
          className="absolute z-10 w-full mt-1 bg-white border rounded shadow-lg"
        >
          {combobox.filteredItems.map((item, index) => (
            <li
              {...combobox.getItemProps(item, index)}
              className="p-2 cursor-pointer hover:bg-gray-100"
            >
              {item.name}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
§ CONTROLLED VS UNCONTROLLED
Decision guide

Guida alla scelta tra componenti controlled e uncontrolled:

Scegli Controlled quando:

Hai bisogno di validazione in tempo reale

Devi sincronizzare lo stato con altri componenti

Vuoi prevenire certi stati/valori

Implementi funzionalità di undo/redo

Gestisci form complessi con dipendenze tra campi

Scegli Uncontrolled quando:

Hai bisogno di performance ottimali (meno re-render)

Il componente è semplice e isolato

Non ti serve validazione immediata

Vuoi integrare con HTML form nativi

Stai migrando da HTML standard a React

typescript
// Esempio comparativo: Input component
import React, { useState, useRef } from 'react';

// Controlled Component
export function ControlledInput({
  value,
  onChange,
  ...props
}: React.InputHTMLAttributes<HTMLInputElement> & {
  value: string;
  onChange: (value: string) => void;
}) {
  return (
    <input
      {...props}
      value={value}
      onChange={(e) => onChange(e.target.value)}
      className="p-2 border rounded"
    />
  );
}

// Uncontrolled Component
export function UncontrolledInput({
  defaultValue,
  onChange,
  ...props
}: React.InputHTMLAttributes<HTMLInputElement> & {
  defaultValue?: string;
  onChange?: (value: string) => void;
}) {
  const ref = useRef<HTMLInputElement>(null);

  const handleChange = () => {
    onChange?.(ref.current?.value || '');
  };

  return (
    <input
      {...props}
      ref={ref}
      defaultValue={defaultValue}
      onChange={handleChange}
      className="p-2 border rounded"
    />
  );
}

// Hybrid Component (supporta entrambi)
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  value?: string;
  defaultValue?: string;
  onChange?: (value: string) => void;
}

export function Input({ value, defaultValue, onChange, ...props }: InputProps) {
  const [internalValue, setInternalValue] = useState(defaultValue || '');
  
  const isControlled = value !== undefined;
  const currentValue = isControlled ? value : internalValue;

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    
    if (!isControlled) {
      setInternalValue(newValue);
    }
    
    onChange?.(newValue);
  };

  return (
    <input
      {...props}
      value={currentValue}
      onChange={handleChange}
      className="p-2 border rounded"
    />
  );
}
Implementation patterns

Pattern di implementazione per componenti che supportano entrambe le modalità.

typescript
// Toggle Switch ibrido
import React, { useState, useEffect, useCallback } from 'react';

interface ToggleSwitchProps {
  checked?: boolean;
  defaultChecked?: boolean;
  onChange?: (checked: boolean) => void;
  disabled?: boolean;
}

export function ToggleSwitch({
  checked: controlledChecked,
  defaultChecked = false,
  onChange,
  disabled = false,
}: ToggleSwitchProps) {
  const [internalChecked, setInternalChecked] = useState(defaultChecked);
  
  const isControlled = controlledChecked !== undefined;
  const checked = isControlled ? controlledChecked : internalChecked;

  const handleToggle = useCallback(() => {
    if (disabled) return;
    
    const newValue = !checked;
    
    if (!isControlled) {
      setInternalChecked(newValue);
    }
    
    onChange?.(newValue);
  }, [checked, disabled, isControlled, onChange]);

  return (
    <button
      role="switch"
      aria-checked={checked}
      disabled={disabled}
      onClick={handleToggle}
      className={`
        relative inline-flex h-6 w-11 items-center rounded-full
        transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2
        ${checked ? 'bg-primary' : 'bg-gray-200'}
        ${disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}
      `}
    >
      <span
        className={`
          inline-block h-4 w-4 transform rounded-full bg-white transition-transform
          ${checked ? 'translate-x-6' : 'translate-x-1'}
        `}
      />
    </button>
  );
}

// Form con gestione ibrida
interface FormData {
  name: string;
  email: string;
  subscribe: boolean;
}

export function UserForm({
  initialData,
  onSubmit,
}: {
  initialData?: Partial<FormData>;
  onSubmit: (data: FormData) => void;
}) {
  // Stato interno per modalità uncontrolled
  const [formData, setFormData] = useState<FormData>({
    name: initialData?.name || '',
    email: initialData?.email || '',
    subscribe: initialData?.subscribe || false,
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      {/* Controlled field - validazione in tempo reale */}
      <div>
        <label className="block text-sm font-medium mb-1">Name</label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
          className="w-full p-2 border rounded"
         